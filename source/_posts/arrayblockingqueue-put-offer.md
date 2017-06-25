title: 关于ArrayBlockingQueue的put和offer方法
date: 2016-05-15 11:01:06
tags: JAVA
categories: 编程开发
---

最近线上出现了一个故障，故障的表现就是服务请响应很慢，依赖方获取不到执行结果，查看调用堆栈，发现所有的操作都阻塞在写日志的地方，这个写日志是先写日志到内存，然后再刷新到其他地方。采用了ArrayBlockingQueue，但是调用了ArrayBlockQueue的put方法
```java
/**
 * Inserts the specified element at the tail of this queue, waiting
 * for space to become available if the queue is full.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        insert(e);
    } finally {
        lock.unlock();
    }
}
```
这里的put方法会等待一个空的位置出来，然后再执行insert，但是系统的请求量非常大，此时一个请求过来后，前面的请求可能还处于等待空位置这一步，此时当前请求获取lock就等待，这样这个业务操作就一直处于获取锁获取不到的场景中了。这是一个真实出现的case，血一般的教训，当时只能不断重启机器来缓解问题。如何彻底解决这个问题，换个API
```java
/**
 * Inserts the specified element at the tail of this queue, waiting
 * up to the specified wait time for space to become available if
 * the queue is full.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {

    checkNotNull(e);
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        insert(e);
        return true;
    } finally {
        lock.unlock();
    }
}
```
换成offer判断返回值为false的情况，不然把业务操作阻塞住。以后在高并发场景下面就不要在使用put这个API了。

