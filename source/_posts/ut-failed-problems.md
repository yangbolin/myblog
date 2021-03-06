title: 单元测试Failed的排查
date: 2014-07-26 15:47:22
tags: 单元测试
categories: 开发杂谈
---

最近各种中间件升级的事情，升级中间件可能会出现很多诡异的问题，此时我们需要定位这些问题是自己升级导致还是其他原因导致，升级中间件可能会出现很多JAR包冲突的问题，此时需要一一排查。

<!-- more -->

前两天升级后遇到这样的问题，系统中的单元测试在eclipse中单独跑没有问题，但是使用mvn test一起跑就会出现大部分单元测试PASS不了，此时查看单元测试FAIL的日志发现获取数据库连接超时，由于本次升级出现了很多JAR包冲突的问题，导致自己一直在怀疑是不是某个JAR包又冲突了导致数据库连接获取超时了？因此排查思路一直停留在JAR包冲突的这个范围内，查了好久，问题依然没有解决，由于自己维护的这个服务化应用非常重要，不能不管单元测试就发布，同时自己也不会放过这个问题，一定要追查到底。

是不是自己的陷入了排查误区了呢？于是把整个代码的变更自己又回顾了一遍，发现最多的就是JAR包版本的升级，引入新的JAR包版本。记得之前自己总结过一句话：要是一个问题排查了好久没有眉目的话，记得去看看那些被你忽略过的细节问题，很可能问题就出现这些被忽略的细节上面。

那到底那些细节被忽略了呢？问题的现象是什么？单个跑UT能够PASS,批量跑UT一部分不能PASS,因为数据库连接获取超时，单个和批量就是这里的细节了，此时自己有一种突然醒悟的感觉，问题就这里了，单个跑只获取一个数据库连接，批量跑会获取多个数据库连接，数据库连接池的问题，不是我升级导致，因此找DBA搞定这个问题。

其实这个问题很简单，发现自己有时候很容易被最近刚出现的问题困扰住，思路一直停留如何最近刚出现的问题上，以后需要注意分析问题出现的本质原因，比如数据连接获取不到其实跟JAR包冲突的关系不是很大，可以说几乎没啥关系，不要受一些最近出现的问题的影响，避免思路被困住。