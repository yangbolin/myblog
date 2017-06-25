title: FutureTask
date: 2014-05-31 20:18:57
tags: [JAVA,并发]
categories: 编程开发
---

在juc的包中有这么一个类FutureTask,我们可以直接用来创建一个该类的实例，然后调用run方法，最后调用get来获取线程的执行结果。

<!-- more -->

看看FutureTask相关的类图

![FutureTask相关的类图](http://bolinyoung.qiniudn.com/futuretask.png)

RunnableFuture一个核心的接口，该接口是Runnable和Future的合体。当然我们也可以创建一批FutureTask然后submit到一个线程池中。

之前不了解juc中的FutureTask这个类，今天看书的时候发现了这个类，算是做个笔记。在以后的开发过程中，需要注意FutureTask这个类的灵活使用。
