---
title: Kubernetes 资源分配
date: 2021-07-12 11:27:21
tags:
---

最近遇到了 Kubernetes 上的 Web 应用响应时间长尾的问题，上网收集资料，分析了一下原因。

- CGroup 节流导致 Web 请求相应时间增加
- 当前执行的 CPU 不断变动
- 核数越多 CGroup 补偿碎片越严重
- 跨越 NUMA 节点

## 请求和限制计算资源

Kubernetes 对资源的细粒度管理功能可以说是吸引传统应用上 Kubernetes 的重要原因之一

我们在定义 Pod 的时候通过以下两个字段可以控制计算资源的分配：

| 字段        | 备注
|------------|----------
| `requests` | 请求的最少资源
| `limits`   | 资源限制

- 当 `requests` 无法满足时，`Pod` 会一直停在 `Pending` 状态，触发 `FailedScheduling` 事件
- 当使用内存资源超过 `limits` 时，如果有其他 POD 需要内存，则使用内存最多的容器会被 `sacrifice`，触发 `OOMKilled` 事件
- 如果不指定 `limits` , 在没有命名空间默认值的情况下可以无限制地使用资源
- 如果指定了 `limits` , 但是没有指定 `requests` , 则 `requests` 值默认为 `limits` 值
- 从分配角度讲 `CPU` 属于可以压缩的资源、内存属于不可以压缩的资源

例如

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

## 命名空间默认值、最小值和最大值

有些 Kubernetes 集群环境下会出现这样的问题：定义 Pod 的时候明明没有给计算资源的限制，但是在实际运行的 Pod 上出现了资源限制的定义。

这很有可能是命名空间的默认值造成的。 通过 `LimitRange` 对象可以设置命名空间资源的默认值、最小值和最大值。

| 关键字                   | 备注
|-------------------------|------------
| default                 | `limit`
| defaultRequest          | `request` 
| min                     | 最小值
| max                     | 最大值
| maxLimitRequestRatio    | 比率
| type                    | 对象

例如

```yaml
apiVersion: v1
kind: LimitRange
metadata:
name: mem-min-max-demo-lr
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

## Pod 服务质量

当系统资源不足时就会有 Pod 遭殃，但是那么多 Pod 怎么知道要干掉哪一个呢。这就和 Pod 的服务质量有关。

查看运行中的 `Pod` 会有一个 `qosClass` 字段，它有以下三种取值：

类型            | 备注
---------------|------------
Guaranteed     | 被保证的，优先级最高，除非超过限制不然不会被干掉
Burstable      | 允许突发，位于中间, 当系统资源不足且没有 `Best-Effort` 级别时
BestEffort     | 尽力保证，优先级最低，当系统资源不足时最先被干掉

需要注意 `qosClass` 的值无法直接设置，而是通过 `requests` 和 `limits` 组合隐式设置

### Guaranteed

- Pod 中的每个容器都要指定 CPU 请求和限制，并且两者相等
- Pod 中的每个容器都要指定内存请求和限制，并且两者相等

### Burstable

- Pod 不符合 Guaranteed QoS 类的标准
- Pod 至少有一个容器具有 CPU 或内存请求

### BestEffort

- 没有 CPU、内存请求和限制

## CPU 管理策略

在现代操作系统的设计下，负载运行可能会不断地迁移到不同的 CPU 核心，Kubernetes 上也是一样。

Kubernetes 提供了 CPU 管理策略来实现负载和 CPU 核的绑定。

通过 kubelet 参数 `--cpu-manager-policy` 来指定 CPU 管理策略。

取值    | 备注
-------|---------------------------------------------
none   | 默认策略，按照 [CFS](https://zh.wikipedia.org/wiki/%E5%AE%8C%E5%85%A8%E5%85%AC%E5%B9%B3%E6%8E%92%E7%A8%8B%E5%99%A8) 来执行调度
static | 允许为节点上具有某些资源特征的 Pod 赋予增强的 CPU 亲和性和独占性

为了将 Pod 绑定到 CPU 需要

- Kubelet 开启 `--cpu-manager-policy static`
- Pod 的服务优先级为 `Guaranteed`
- Pod 的 CPU 资源为整数

例如，下面的 Nginx Pod

```
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "2"
      requests:
        memory: "200Mi"
        cpu: "2"
```

在运行时可独占两颗 CPU

## NUMA 和拓扑管理策略

NUMA 是系统优化的一个常用思路，它是伴随着多处理器出现的一个问题。

> 非统一内存访问架构（英语：Non-uniform memory access，简称NUMA）是一种为多处理器的电脑设计的内存架构，内存访问时间取决于内存相对于处理器的位置。

常见的硬件有

- CPU
- Memory
- GPU
- NIC

安装 `numactl` 后可以通过下面的命令查看主机的 NUMA 相关信息

`numactl --harware`

通过下面的命令查看 NUMA 的统计信息

`numastat`

```
numa_hit       | Number of pages allocated from the node the process wanted.
numa_miss      | Number of pages allocated from this node, but the process preferred another node.
numa_foreign   | Number of pages allocated another node, but the process preferred this node.
local_node     | Number of pages allocated from this node while the process was running locally.
other_node     | Number of pages allocated from this node while the process was running remotely (on another node).
interleave_hit | Number of pages allocated successfully with the interleave strategy.
```

以上指标均可以通过 Telegraf 监控

```
[[inputs.kernel_vmstat]]
```

`--cpu-manager-policy static --topology-manager-scope pod --topology-manager-policy single-numa-node`

## 参考

- https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-memory-resource/
- https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-cpu-resource/
- https://kubernetes.io/zh/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/
- https://kubernetes.io/zh/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/
- https://kubernetes.io/zh/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/
- https://kubernetes.io/zh/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/
- https://kubernetes.io/zh/docs/tasks/configure-pod-container/quality-service-pod/
- https://kubernetes.io/zh/docs/tasks/administer-cluster/cpu-management-policies/
- https://kubernetes.io/zh/docs/tasks/administer-cluster/topology-manager/
- https://kubernetes.io/docs/tasks/administer-cluster/memory-manager/
- https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/resource-qos.md