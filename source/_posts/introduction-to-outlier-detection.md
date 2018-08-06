---
title: "异常指标检测"
date: 2016-12-28 12:00:00
---

[源码](https://github.com/longtian/cache_attack_in_javascript)

## 目前有两种算法

### MAD 中位数绝对值偏差

对应的 JavaScript 实现

[MAD in mathjs](http://mathjs.org/docs/reference/functions/mad.html)

### DBSCAN 基于密度的聚类算法

对应的 JavaScript 实现

[DBSCAN in JavaScript](https://github.com/uhho/density-clustering#dbscan-1)

## 准确度

要准确的判定一个指标是否异常是很难的，取决于很多因素:

- 数据本身的特征
- 窗口大小
- 节假日等突发情况
- 噪点

上面的两种算法都是经过 Datadog 工程师[精心挑选](https://www.youtube.com/watch?v=mG4ZpEhRKHA)的，有这样的特点：

- 鲁棒性 避免噪点的干扰
- 可用性 配置参数尽量少

此外各算法的参数还需要不断地尝试，才能找到一个最佳的配置。

## 参考资料

[Detecting outliers and anomalies in realtime at Datadog - Homin Lee (OSCON Austin 2016)-Youtube](https://www.youtube.com/watch?v=mG4ZpEhRKHA)

[MAD(Mean 变体)-Khanacademy](https://www.khanacademy.org/math/6th-engage-ny/engage-6th-module-6/6th-module-6-topic-b/v/mean-absolute-deviation)

[DBSCAN in scikit-learn](http://www.cnblogs.com/pinard/p/6217852.html)

[Visualizing DBSCAN Clustering](https://www.naftaliharris.com/blog/visualizing-dbscan-clustering/)
