title: CountDownLatch和CyclicBarrier
date: 2014-05-11 21:11:11
tags: [JAVA,并发]
categories: 编程语言
---
####一.概述
在编写并发程序的时候，我们经常遇到这样的事情，就是当N个线程都走到某一步后再继续做其他事情，都走到某一步包含这N个线程都运行结束这种场景。我们可以使用线程同步的思想来解决这类问题，在C语言中中我们经常使用信号量来解决此类问题。当然要是你自己使用线程同步来解决个问题，你需要做很多事情，不过jdk提供了一些封装好的类帮助我们解决此类问题，这里我们来看如何使用juc包中提供的CountDownLatch和CyclicBarrier来解决这类问题。

<!-- more -->

####二.使用CountDownLatch
![CountDownLatch](http://bolinyoung.qiniudn.com/CountDownLatchTest.png)

* 1.在主线线程我们创建一个CountDownLatch实例，其计数值为2，这样必须要执行两countDown()方法，await()方法才能被唤醒。
* 2.在ComputeThread中等线程的计算完成后我们执行countDown()方法，这里只是为了演示，一般我们需要把countDown()方法的执行写在finally代码块中。
* 3.在main方法中我们执行await()方法等待CountDownLatch的计数值变为0后，主线程再继续向下执行。

这样我们使用CountDownLatch就可以保证这两个线程把核心的计算做完之后，主线程再继续向下，因此主线程继续向下的前置条件是等这两个线程把该做的事情做完。

上述代码的运行结果如下：

> compute thread 1 is running...
> compute thread 2 is running...
> All compute Thread has finished...


三.使用CyclicBarrier
![CyclicBarrier](http://bolinyoung.qiniudn.com/CyclicBarrierTest.png)

* 1.在主线程中我们创建一个CyclicBarrier实例，并且传递一个计数值2,同时我们以匿名内部类的方式提供一个barrier,当所有的线程都达到某个点的时候，我们就执行这个barrier。
* 2.在没有线程中我们执行await方法等待其他线程执行，当计数值减少到0后就表示相关的线程都到barrier前面，此时barrier就开始执行，如下图所示
![线程屏障](http://bolinyoung.qiniudn.com/barrier4thread.png)
此时要是线程Thread2先执行完了，它必须等线程Thread2把相关的操作也执行完。

上述代码的运行结果如下：

> Horse Thread1 is running...
> Horse Thread2 is running...
> All threads have arrived here...


####四.CountDownLatch和CyclicBarrier区别
* 1.CountDownLatch一个线程等其他N线程执行完相关的操作
* 2.CyclicBarrier N线程中每个线程都需要等待其他N-1线程执行完
* 3.CyclicBarrier的计数可以复用，CountDownLatch不可以复用