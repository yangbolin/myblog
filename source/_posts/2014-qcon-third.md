title: 2014.04.27 Qcon北京第三天总结
date: 2014-04-30 11:03:53
tags: Qcon
categories: 技术交流
---

####一.概述
2014Qcon北京第三天我主要关注了一个主题，尖端之上的Java,分享者主要讲了一些自己公司中如何使用Java开进行日常开发，这个主题也比较火。下午关于文化科研相关的演讲，我没怎么仔细听。

<!-- more -->

####二.你应该更新的Java知识

这位分享者已经有10年的开发经验了，他的博客也很有名[dreamhead](http://dreamhead.blogbus.com/),有空可以去翻翻。

10多年来，软件的开发方式没有发生本质上的改变。
Rails 约定胜于配置。
多核和函数式编程，MapReduce的思想来自函数式编程。
每年都去学一门新的语言。

* clean code
整洁代码，函数越短越好，写代码不要太随便，不是说满足需求，实现功能就行，要能优雅的满足需求，实现功能。
重构，rework,有时间读读这本书，[rework](http://book.douban.com/subject/3889178/)
自定义注解，使用annotation来简化代码

* 函数式编程
懒加载
* DSL
领域特定语言，可以看看相关的书籍
设计并实现自己的DSL
* 新风格程序库
Guava 
Joda-Time 
Hamcrest 
Mockito 
DropWizard 
Junit 
DSL
easyMock
JMock

把getter/setter改造成更加良好的DSL
有时间去看看Scala

####三.API单位误解造成严重的故障

在socket编程中有一个setLiner方法，我记不清是那个类的了，单位值是秒，但是被开发误以为是毫秒，导致了一个线上故障，出现线上故障的时候注意保留现场，方便后续问题的排查。

tomcat和jetty都有线程池大小，每个应用都会配置一个活跃线程数。

线程dump jstack
内存dump jmap 使用MAT分析

-XX:+PrintFlagsFinal 输出所有参数的名称和默认值,也可以使用jinfo -flags来查看值，使用jinfo -flag来设置值。
-XX:+TraceClassLoading 跟踪类加载信息。

应用无响应的排查思路，先看一下应用在干什么，使用jstack
jstack -l 打印锁状况
jstack -m 把native的堆栈也打印出来

static代码块初始化的时候，会在类上加上一把锁。

内存相关
-XX:+HeapDumpOnOutOfMemory JVM在OutOfMemory的时候，会拍摄一个堆转储快照，同时转存到一个文件中。
线上应用可以带上输出GC日志的JVM参数-XX:+PrintGC，对应用本身不会有太大的影响。

jmap -histo:live 查看JVM中存活的实例。
jmap -dump:file dump JVM的内存。
dump出来的内存文件大小和堆大小差不多大，内存太大会导致内存的dump很耗时。

gcore 命令
generates a core file for the process specified by its process ID, pid. By default, the core file is written to core.pid, in the current directory。
可以使用 gcore pid 来获取一个进程的core文件。

使用MAT分析dump下来的内存的时候，没法定位到具体的代码，这时候可以结合BTrace来使用。

[gperftools](http://www.cnblogs.com/caosiyang/archive/2013/01/25/2876244.html)

ulimit
cat /proc/pid/limits
这里的pid是进程ID

CPU利用率
top -H && jstack
1.使用top命令查看占用cpu最高的线程ID
2.把线程ID转换成十六进制
3.在jstack出来的内从找线程ID相关的线程，看看线程在做什么事情。

注意:
top和jstack之间有时间的误差，就是说我们使用top看到的线程可能在jstack dump出来的线程堆栈的找不到。

Java进程退出
1. 打开core dump文件看看
2. 使用gdb调试
3. 使用dmesg | grep -i kill 看看是不是os把java进程kill掉

####四.去哪儿网的java开发生态环境
* 发布相关
有相同的类，就不允许发布，解决依赖冲突的问题。

* 架构相关
在实际的业务逻辑处理中，有时候我们做完一些业务上的操作后，需要发消息给其他模块或系统，在一些高要求的系统中，我们需要保证业务操作和发消息的事务，为了达到这么目的，在数据库中建立一张消息表，发消息的时候往数据库中插入一条记录，这样往消息表中插入数据的操作和业务表的操作在数据库层面形成事务，最后又有一个后台线程，不断去扫描消息表，然后把消息发送出去。

为了保证消息和业务逻辑之间的事务，他们借助一张消息表来实现，每次发消息都先往消息表中写一条记录，我个人感觉先尝试去发消息，要是消息发送失败再往消息表中写一条记录，然后使用一个后台线程去轮寻消息表，把这些发送失败的消息再发送出去，不太理解他们为什么每次发消息都往消息表中插入一条记录。

* mock平台
很多网站架构中都有mock平台这一部分，这样可以方便测试环境覆盖更多的业务逻辑分支。
* 自动化测试
产品经理要会写测试用例，这是他们的要求，在他们看来产品经理必须对产品的边边角角都有一个非常清晰的理解，他们通过这样的方法来保证这个点。
* codeReview
去哪儿网还是比较重视CodeReview的
* 监控体系

####五.大众点评零压力中间件部署
大众点评需要解决的问题是降低中间件升级的成本，中间件的升级不需要应用方来修改相关的配置或者代码，由中间件团队自己在生产环境部署升级，实现平台也业务的分离。

每次中间件的升级，中间件团队把新版本的中间件推送到线上，然后重新部署线上的应用，就可以完成中间件的升级，这时候新版本和老版本的中间件都会存储在线上应用中，他们提供一个规则的配置文件，应用会根据规则的配置文件来决定使用中间件的那个版本。

技术点：
重写tomcat的WebXmlLoader和WebApplicationLoader

其实这种升级中间件的思路很好，但是他们做的不彻底。比如说今天要升级中间件A了，中间件团队在测试环境找QA测试好之后，便自己直接去线上部署，并且线上运行没啥问题，但是线上应用pom依赖中中间件的版本还是老版本，这个老版本还是需要相应应用的开发自己去改，在他们看来只要应用中依赖的版本不要过低，都没啥问题的。

####六.搜狗商业广告平台java生态演化之路

这个讲的有点虚，没啥实质性的内容，什么sso啥的都是一笔带过。他们也利用了Thrift，关于Thrift和Restful我不太了解。最后还提到了他们利用Zookeeper的推送机制来实现数据的推送,Zookeeper数据节点发生变化的时候，它会发起相关的通知。
