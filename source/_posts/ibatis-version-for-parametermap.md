title: IBatis不同版本对parameterMap的处理
date: 2014-07-23 19:58:04
tags: IBatis
categories: 编程开发
---

####一.概述
今天在升级一个监控的中间件时，发现必须升级ibatis的版本到2.3.4,原来ibatis的版本是2.3.0,升级后，发现sqlmap的解析出问题了，对于parameterMap这个标签解析出错，原因是parameterMap对应class没法实例化，为什么原来没有问题，升级后就有问题了呢？这时候只能怀疑版本的问题,2.3.4和2.3.0在解析parameterMap时有所不同。

<!-- more -->

下面是我们应用中一份sqlmap中写的parameterMap
![parameterMap](http://bolinyoung.qiniudn.com/parameterMap.png)

其实这个parameterMap在这份配置文件中压根就没有被使用过，TA-COACH-CONFIG-PARAM这个class在当前的classpath中显然不存在，同时也不是某个class在sqlmap中的别名，因此在解析的时候会报TA-COACH-CONFIG-PARAM这个class找不到。

#####二.到底那里不一样
那具体到ibatis的源代码中，那些代码发生了变更呢？我们首先找到解析sqlmap的类即SqlMapParser这个类，先看看2.3.0中这个类解析paramterMap的行为
![2.3.0](http://bolinyoung.qiniudn.com/parameterMap230.png)
我们看到在2.3.0这个版本中拿到parameterClassName后，尝试去实例化这个类型的实例，此时要是这个类找不到就出现Exception，但是该版本中没有处理这个异常，就是说这个异常默默地被吞了。

接下来再看看2.3.4中SqlMapParser解析parameterMap的行为
![2.3.4](http://bolinyoung.qiniudn.com/parameterMap234.png)
其实在2.3.0中作者已经意识到这个事情了，只是写了TODO，在2.3.4中正式开始抛出异常了。

综上所述：
在ibatis的配置的文件中，要么使用类的权限定名，要么使用类的别名，要是一个class在classpath中不存在，但是你在sqlmap中使用了，你就等着翻源码查问题吧。还好上面的那个parameterMap形同虚设，我可以放心地删了它。