title: JAVA线程中join,wait,notify&&notifyAll
date: 2014-06-03 22:19:16
tags: [JAVA,并发]
categories: 编程语言
---

####一.概述
在编写多线程程序的时候，我们经常需要考虑线程之间的同步，比如我们使用join可以让某一线程执行结束，使用wait可以让某一线程在某个地方等待，使用notify或者notifyAll可以唤醒处于等待状态的线程。这里我们来看看这几个线程自带方法的使用以及相关的分析。

<!-- more -->

####二.join()的使用
join()方法是Thread类的，在Thread类中提供三种join()方法
```java
// 设置毫秒级别的等待
public final synchronized void join(long millis) 
    throws InterruptedException {//...}
// 设置毫秒级别，纳秒级别的等待
public final synchronized void join(long millis, int nanos) 
    throws InterruptedException {//...}
// 持续等待，没有时间限制
public final void join() throws InterruptedException {//...}
```

如果我们写下面的代码
```java
Thread t1 = new ...
t1.start();
t1.join();
System.out.println("t1 has finished...");
```
此时控制台的输出一定是在线程t1执行结束后，从这点我们可以利用join来控制多个线程直线的执行顺序，比如要想线程t2在线程t1执行结束后执行，我们在调t2的start方法之前调用t1的join方法即可。

####三.wait()&&notify()&&notifyAll()的使用
wait(),notify(),notifyAll()这三个方法是任何一个java对象都具有的方法。
在某一对象上调用wait()方法，当前线程就会在该对象上处于等待状态。
在某一对象上调用notify()方法，就会唤醒一个在当前对象上处于wait状态的线程，要是当前对象上有N个线程处于wait状态，就会任意选取一个唤醒。
在某一对象上调用notifyAll()方法，就会唤醒在某一对象上处于wait的所有线程。

注意wait()和notify()的时候要使用相同的Object。
####四.绿色线程&&本地线程
最近在搜索的时候发现了一个绿色线程的概念，很好奇，继续搜索了一把，发现绿色线程在jdk1.1的时候存在，后面就给干掉了，绿色线程的意思就是这个线程只是一个JVM可调度运行的任务，不会对应一个OS层面的上的线程。本地线程必须对应一个OS层面的线程。
####五.总结
线程的同步我们一般不太会使用join,wait,notify以及notifyAll等来同步，直接使用juc包中的一些类，但是我们需要明白这些方法的具体含义。
