title: JAVA线程模型
date: 2014-07-26 19:13:14
tags: 并发
categories: 编程开发
---

我们在JAVA代码中创建一个线程，这线程在系统中会经历下面5个阶段：
New->Runnable->Blocked->Running->Dead

<!-- more -->

这里我们来详细看一下这几个阶段的演变以及转换
![线程状态转换](http://bolinyoung.qiniudn.com/java-thread-model.png)

* New
表示这个线程对象刚刚被创建，它具备线程的一些特性，但是系统没有给它分配资源。

* Runnable
表示这个线程处于就绪状态，要是能获得CPU的话，线程马上就能运行起来。

* Blocked
一个正在运行的线程由于某些原因不能继续运行，它将进入阻塞状态。比如线程对象执行suspend(),sleep()等阻塞类型的方法后，线程就会进入BlockedPool，当线程调用resume()方法或者自动苏醒后，线程会进入Runnable状态;线程因为要执行synchronized同步代码块需要获取锁，但是当前要获取的锁被其他线程所占有，此时线程会进入LockPool，要是线程获取到这把锁后，线程又会进入到Runnable状态;线程执行了某个对象的wait()方法，此时线程会进入到WaitPool，当相应对象上的notify方法被调用后，线程就会进入Runnable状态。

* Running
处于Runnable状态的线程要是能获取到CPU的话，马上就变成Running状态

* Dead
线程运行结束，或者线程被interrupt，或者线程被stop，线程的生命周期也就结束了。

JVM在执行某个方法的时候，会创建一个方法执行栈帧(Frame),这个栈帧包含局部变量区和操作数栈，在局部变量区的第一个位置上存储当前方法所属的实例对象。

