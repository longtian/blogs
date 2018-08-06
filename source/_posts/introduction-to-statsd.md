---
title: "StatsD 简介"
date: 2016-05-04 12:00:00
---

[源码](https://github.com/longtian/introduction-to-statsd)

> 介绍 StatsD 的发展，协议，最后附带有一个假想的例子。

自主开发一套监控系统可以从这三个部分分别着手：数据采集，数据存储和数据展示。

对于有代码经验的开发和运维人员，数据采集是很容易解决的问题，例如要统计一个方法的执行时间，只需要记录方法进入和退出的时间差即可。
而对于后台架构中常见的各种组件如 Redis 和 MySQL 等，它们往往各自提供了数量可观的指标接口，使得运维人员可以快速的获取组件的运行状况。

数据存储曾经是比价复杂的一块，首先就是数据结构的设计，此外还要考虑快速的检索，统计操作等。
好在最近有很多兴起的时序数据库，如 InfluxDB 和 OpenTSDB，它们都对存储时间序列型的数据有很强的适用性。
数据展示可以采用开源的方案例如 Grafana 。

**StatsD 的使用场景**

把数据推送到存储模块的过程中，有的时候没有必要把所有采集到的数据都存起来。
典型的场景是 Web 服务器，当请求达到每分钟上万条时，把每条请求的响应时间都存入数据库是不现实也没有必要。

这个时候可以考虑设计一个缓冲模块，来实现数据的累加，平均值，中间值计算等。
StatsD 正是这样一个实现数据中转的工具，它能够完成简单但是非常实用的统计步骤。
另一方面，由于往 StatsD 写入数据的时候使用的是 UDP 协议，所以对被检测的系统的影响可以控制的很小。

## 安装 StatsD 服务

StatsD 安装指南

https://github.com/etsy/statsd#installation-and-configuration

装好 StatsD 以后，还需安装 InfluxDB 或者 OpenTSDB，由于篇幅的关系不再展开。

本文推荐使用 CloudInsight 监控系统，如果已经有 OneAPM 的账号，安装只需要 1 分钟左右。

http://docs-ci.oneapm.com/quick-start/

# StatsD 协议

StatsD 的协议其实非常简单，每一行就是一条数据。

```
<metric_name>:<metric_value>|<metric_type>
```

以监控系统负载为例，假如某一时刻系统 `1分钟的负载` 是 0.5，通过以下命令就可以写入 StatsD。

```sh
echo "system.load.1:0.5|g" | nc 127.0.0.1 8251
```

结合其中的 `system.load.1:0.5|g`，各个部分分别为：

- 指标名称 metric_name  = system.load.1
- 指标的值 metric_value = 0.5
- 指标类型 metric_type  = g(gauge)

**指标名称**

指标命名没有特别的限制，但是一般的约定是使用点号分割的命名空间。

**指标的值**

指标的值是大于等于 0 的浮点数。

**指标类型**

- [gauge](gauge-标量)
- [counter](#counter-计数器)
- [ms](#ms-时间)
- [set](#set-集合)

StatsD 封装了一些最常用的操作，例如周期内指标值的累加、统计执行时间等，因此对指标的值分成以下几种：

#### gauge 标量

标量是任何可以测量的一维变量。例如人的身高，体重、某一时刻北京市 PM2.5 的值等。

#### counter 计数器

如果某个变量没有办法一下子测量出来，这时就可以使用计数器，分步累加。

例如要统计某个接口的流量

```
ent0.traffic:100|c
```

在每个周期里 StatsD 会自动累加流量的值，并把平均值传给后端。

#### ms 时间

记录执行时间。请求响应时间是 12 ms，写入到 StatsD 服务器的方法是：

```
echo "request.time:12|ms" | nc 127.0.0.1 8251
```

StatsD 会自动计算以下指标：

```
request.time.avg
request.time.count
request.time.max
request.time.mean
...
```

#### set 集合

统计变量不重复的个数。例如：当前在线的用户

'echo "online.user:9587|s" | nc 127.0.0.1 8251'

'echo "online.user:9588|s" | nc 127.0.0.1 8251'

# StatsD 实例

以做一个完美的煎饼果子的过程为例，参数和类型的对应关系如下：

**计数器**

- 鸡蛋
- 薄脆
- 面粉

**标量**

- 油温
- 辣度

**集合**

- 口味

**计时器**

- 做每一个煎饼果子的时间

## 运行代码

```
git clone git@github.com:wyvernnot/introduction-to-statsd.git
cd introduction-to-statsd
npm install
npm run-script pipe
```

```
http://localhost:8080
```

## 参考

- https://github.com/etsy/statsd/blob/v0.7.2/docs/metric_types.md

- https://github.com/etsy/statsd/wiki#client-implementations

- http://docs.datadoghq.com/guides/dogstatsd/#datagram-format
