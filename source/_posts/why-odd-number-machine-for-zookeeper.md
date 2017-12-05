title: 为什么zookeeper集群个需要奇数台机器
date: 2014-05-17 15:39:29
tags: zookeeper
categories: 分布式
---

我们通常使用zookeeper集群来协同管理另外一个集群，比如HBase集群，一个HBase集群必然对应一个zookeeper集群。在部署zookeeper集群的时候，我们为什么选择奇数台机器呢？

<!-- more -->

关于zookeeper集群有下面几点说明

* 1.集群中机器数目越多越稳定
* 2.集群通过选举的方式来选出集群的leader，要是有一半以上的机器同一某个机器成为leader,那么这个机器就编程集群中的leader
* 3.要是集群中有一半以上的机器挂掉，整个集群就会挂掉

对于2N和2N+1台机器，选出leader的时候都需要有N+1票，就是说N+1台机器投票给某一台机器，因此2N和2N+1台机器构成的集群在投票过程中所投票数一样，但是2N+1台机器构成的集群稳定。同样2N台机器中挂掉N台的概率是50%，但是2N+1台机器中挂掉N太的概率小于50%，也就是说2N+1台机器构成的集群比较稳定。

综上所述：
zookeeper集群选择2N+1台机器是比较合理的。