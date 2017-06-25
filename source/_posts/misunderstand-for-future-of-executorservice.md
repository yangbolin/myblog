title: 关于ExecutorService的Future的误解
date: 2014-05-17 20:54:15
tags: [JAVA,并发]
categories: 编程语言
---

####一.概述
在多线程编程中我们经常使用ExecutorService的Future来实现等待多个线程返回的需求。此时需要调用ExecutorService的相关方法来实现，比如

<!-- more -->

```java
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
```

调用这个方法后，遍历返回的List就能获取到本次新增tasks的返回值，要是某个task执行时间较长，调用Future的get方法就会阻塞，当然你也可以选择使用能设置超时时间的get方法。

Future的get方法，一个可以设置超时时间，一个不能设置超时时间。
```java
V get() throws InterruptedException, ExecutionException;
V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```

这样我们就能等所有线程都返回后，然后继续做相关的计算逻辑。

####二.常常被误解的地方

我们在一个线程池A上调用下面的方法
```java
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
```
假设在调用这个方法之前线程池中有M个线程在RUNNING,调用这个方法后又往线程池中增加了N个线程，然后遍历返回的List,此时返回的List中只会包含后面加入的N个线程，而不会包含前面已经存在的M个线程，因此List的大小是N，不是M+N,经常会有人误以为是M+N。因此当遇到需要N个线程都返回后才继续向下的场景时，就放心大胆的使用Future吧。
