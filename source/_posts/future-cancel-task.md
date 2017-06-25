title: 通过Future来取消任务
date: 2014-05-31 19:01:51
tags: [JAVA,并发]
categories: 编程开发
---

####一.概述
在多线程程序中，我们经常使用Future来获取线程的执行结果，我们想线程的执行是异步的，因此要想获取线程执行结束后的结果，我们就需要等线程执行结束，这是一种主动获取线程执行结果的方法，当然你也可以让线程拿到结果后通知你。在使用Future的时候我们经常会设置一个超时时间，要是过了这个超时时间，线程的结果还没返回，那么我们经常就会忽略这个结果，但是这个获取结果的线程怎么办呢？让他自生自灭？我看代码中经常有这种写法，让这样的线程自生自灭。

<!-- more -->

####二.通过Future来取消线程
要是相应的线程在规定的时间内没有返回结果，我们就把这个线程给终止了，不要让它自生自灭，因为此时线程的执行已经没有意义了，要是我们不管它，让它自生自灭就等于浪费系统资源，这个点很容易被忽略，说实话自己之前也经常忽略，直到今天看书的时候才发现。下面我们来看看使用Future的良好编程习惯：

```java
public static void timedRun(Runnable r, long timeout, TimeUnit unit) throws InterruptedException {
    Future<?> task = taskExec.submit(r);
    try {
        // 设置任务执行的超时时间
        task.get(timeout,unit);
    } catch(TimeoutException e) {
        // 超时异常
    } catch(ExecutionException e) {
        // 执行异常
    } finally {
        // 如果任务已经结束，那么执行取消不会带来任何影响，如果任务正在执行，那么将会被中断
        task.cancel(true);
    }
}
```

使用Future的时候，记得不要让没有用的任务自生自灭。