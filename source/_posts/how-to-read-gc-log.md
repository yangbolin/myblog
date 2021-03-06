title: 如何读懂JVM GC日志
date: 2014-07-01 23:55:05
tags: JVM
categories: JVM学习
---
我们经常会在线上观察JVM运行时GC日志，其实GC日志中有很多信息可以挖掘，比如下面的GC日志信息

> [GC [DefNew: 34538K->2311K(36352K), 0.0232439 secs] 45898K->15874K(520320K), 0.0233874 secs]      
> [Full GC [Tenured: 313563K->15402K(483968K), 0.2368177 secs] 
343563K->18402K(520320K), [Perm : 28671K->28635K(28672K)], 0.2371537 secs]

<!-- more -->

观察上面的GC日志，我们发现出现了一次YGC和一次FGC,JVM的内存是分代的，有老年代，新生代，还有永久代。一般我们创建的对象先会进入新生代，要是经过N次YGC后，这个对象还存在，那么这个对象就晋升到老年代，这个N是可以配置的。永久代主要存储JVM字节码以及相关的类信息。

* 关于YGC日志的解释：
新生代的总大小是36352K,当新生代使用到34538K时发生了一次YGC,此时新生代的占用从34538K变到2311K，同时我们也能看到整个堆内存的变化，整个堆内存大小是520320K，当使用到45898K时发生了一次YGC,堆内存的使用由45898K变化到15874K。后面的时间是指内存回收的耗时。

* 关于YGC日志的解释：
老年代总大小为483968K，当老年代使用到313563K，发生了一次FGC,老年代的使用由313563K变到15402K。堆的总内存大小为520320K，当使用到343563K发生了一次FGC,堆内存的使用由343563K变到18402K。永久代的总大小为28672K，当永久代内存使用到28671K时发生了一次FGC，永久代使用的内存由28671K变到28635K。后面的时间表示内存回收的耗时。