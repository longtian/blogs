---
title: JVM 对容器化支持的参数
date: 2021-04-13 17:23:31
tags:
---

## 背景知识

阅读本文你需要了解

- JVM 的启动参数
- 容器的启动参数

### JVM 启动参数

参数   | 备注            
------|----------------
`-Xms`| MiniumHeapSize
`-Xmx`| MaxHeapSize
`-Xss`| StackSize

在容器化之前一般由运维工程师根据部署环境机器的内存大小来调整，并写在启动脚本里

### 容器启动参数

在使用 k8s 编排容器的时候通常通过定义 resources 字段来限制容器使用的资源

```yaml
resources:
  limit:
     cpu: 1000m
     memory: 1G
```

为了方便验证，我们直接用本地 docker 来实验，docker run 命令的一些参数

参数                         | 备注
----------------------------|---------------------------------------------------------------
--cpus                      |     Number of CPUs
--cpu-period int            |     Limit CPU CFS (Completely Fair Scheduler) period
--cpu-quota int             |     Limit CPU CFS (Completely Fair Scheduler) quota
--cpu-rt-period int         |     Limit CPU real-time period in microseconds
--cpu-rt-runtime int        |     Limit CPU real-time runtime in microseconds
--cpu-shares int            |     CPU shares (relative weight)
--memory                    |     Memory limit

## 问题

容器化的 JVM 程序会遇到这几个问题

- 问题 1： 超过了容器的资源限制被 OOM 机制杀掉
- 问题 2： 在核数比较高的机器上 JVM 频繁 GC

### 问题 1

JDK 8u191 之前这个问题很常见，主要是由于 JVM 获得的最大内存是主机的最大内存而不是容器的最大可用内存，解决方法是修改启动逻辑。
让 JVM 能够获取到正确的最大可用内存

### 问题 2

ParallelGC 线程数如果过高会导致 GC 过于频繁，和问题1 类似，这是由于 JVM 获得的可用核数不正确导致的。

为了解决这两个需要同时修改容器的资源限制和 JVM 的启动参数，并使之相匹配。
当然也有一种不那么笨的做法是引入一个动态的启动脚本，先根据容器的实际资源限制算出对应的 JVM 参数再去启动 JVM。
JDK 社区已经注意到以上这几个容器化常遇到的问题并在新的 JDK 版本里解决了，由于 JDK8 的超长待机时间，以下便以 JDK8 为例

版本     | GA 时间                                                                           |  验证镜像              | 容器化支持
--------|-----------------------------------------------------------------------------------|----------------------|------
`8u191` | [2018-10-16](https://www.oracle.com/java/technologies/javase/8u191-relnotes.html) | openjdk:8u191-alpine | 支持
`8u131` | [2017-04-18](https://www.oracle.com/java/technologies/javase/8u131-relnotes.html) | openjdk:8u131-alpine | 实验性支持
`8u121` | [2017-01-17](https://www.oracle.com/java/technologies/javase/8u121-relnotes.html) | openjdk:8u121-alpine | 不支持

## 识别 CPU 资源

实验： 通过 `ParallelGCThreads` 是否符合预期来验证 JDK 对 CPU 资源的识别是否正确

```
java -XX:+PrintFlagsFinal -XX:+UseParallelGC -version | grep -i ParallelGCThreads
```

### 8u191 之前不支持

```
docker run --cpus 1 --rm openjdk:8u121-alpine java -XX:+PrintFlagsFinal -XX:+UseParallelGC -version | grep -i ParallelGCThreads
```

结果不符合预期，结果为主机核数

### 8u191 支持识别可用 CPU 核数

```
docker run --cpus 1 --rm openjdk:8u191-alpine java -XX:+PrintFlagsFinal -XX:+UseParallelGC -version | grep -i ParallelGCThreads
```

结果符合预期，结果为 1

### 8u191 关闭自动识别可用 CPU 核数

在分到 4 个 CPU 核数的情况下，指定可用 CPU 核数为 1

```
docker run --cpus 4 --rm openjdk:8u191-alpine java -XX:+PrintFlagsFinal -XX:+UseParallelGC -XX:ActiveProcessorCount=1 -version | grep -i ParallelGCThreads
```

结果符合预期，结果为 1

## 识别 Memory 资源

实验： 通过查看 JVM 运行时的参数来验证 JVM 对 Memory 资源的识别是否正确

`java -XshowSettings:vm -version`

### 8u121 不支持

```
docker run -m 100MB --rm openjdk:8u121-alpine java -XshowSettings:vm -version
```

结果不符合预期，结果为主机内存的 25%，这是因为这个版本 JDK 的 MaxHeapSize 计算公式为：

```
MaxHeapSize = MaxRAM / MaxRAMFraction
```

其中的 `MaxRAM` [来自于主机]()

### 8u131 实验性支持

直接使用 8u131 版本的 JDK 运行

```
docker run -m 100MB --rm openjdk:8u131-alpine java -XshowSettings:vm -version
```

结果不符合预期，说明 8u131 默认没有开启对容器化的支持，要开启需要加上实验性参数

```
docker run -m 100MB --rm openjdk:8u131-alpine java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XshowSettings:vm -version
```

结果符合预期

### 8u191 默认支持

使用 8u191 版本运行

```
docker run -m 100MB --rm openjdk:8u191-alpine java -XshowSettings:vm -version
```

结果符合预期

### 8u191 关闭自动识别最大可用内存

```
docker run -m 100MB --rm openjdk:8u191-alpine java -XX:-UseContainerSupport -XshowSettings:vm -version
```

结果符合预期，结果为主机内存的 25%

## JVM 容器化相关参数

```
docker run -m 100M --rm openjdk:8u191-alpine java \
  -XX:+UnlockExperimentalVMOptions \
  -XX:+UnlockDiagnosticVMOptions \
  -XX:+PrintFlagsFinal \
  -XX:+UseParallelGC -version | grep -i 'cgroup\|container\|parallelgcthreads'
```

### ActiveProcessorCount

> Specify the CPU count the VM should use and report as active

8U191 引入

### PrintContainerInfo

必须使用 UnlockDiagnosticVMOptions 开启

```
docker run -m 100MB --rm openjdk:8u191-alpine java -XshowSettings:vm -XX:+UnlockDiagnosticVMOptions -XX:+PrintContainerInfo -version
```

### UseContainerSupport 

> Enable detection and runtime container configuration support

`8U191` 引入

### PreferContainerQuotaForCPUCount

> Calculate the container CPU availability based on the value of quotas (if set), when true. Otherwise, use the CPU shares value, provided it is less than quota.

### UseCGroupMemoryLimitForHeap   

> Use CGroup memory limit as physical memory limit for heap sizing

`8U131` 引入

在 JDK11 中已移除，被默认开启的 `UseContainerSupport` 代替

### MinRAMFraction  

> Minimum fraction (1/n) of real memory used for maximum heap  size on systems with small physical memory size. 
> Deprecated, use MinRAMPercentage instead

`8U131` 引入 `8U191` 移除

### MaxRAMFraction

> Maximum fraction (1/n) of real memory used for maximum heap size.                                                          \
> Deprecated, use MaxRAMPercentage instead

`8U131` 引入 `8U191` 移除

### InitialRAMPercentage

> Percentage of real memory used for initial heap size

`8U191` 引入

### MaxRAMPercentage

> Maximum percentage of real memory used for maximum heap size

MaxRAMPercentage 在可用内存大于 250MB 时生效

```
docker run -m 1000MB --rm openjdk:8u212-alpine java -XshowSettings:vm -XX:MaxRAMPercentage=50.0 -XX:MinRAMPercentage=50.0 -version
```

### MinRAMPercentage

> Minimum percentage of real memory used for maximum heap size on systems with small physical memory size

MaxRAMPercentage 和 MaxRAMPercentage 都是用来计算 MaxHeapSize 的

MinRAMPercentage 在可用内存小于 250MB 时生效 （实际很少除非 IoT 设备）

```
docker run -m 200MB --rm openjdk:8u212-alpine java -XshowSettings:vm -XX:MaxRAMPercentage=50.0 -XX:MinRAMPercentage=50.0 -version
```

### 参考

- https://dzone.com/articles/difference-between-initialrampercentage-minramperc