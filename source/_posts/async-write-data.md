title: 一个数据产品中异步写数据的实现思路
date: 2014-04-12 15:20:21
tags: 并发
categories: 编程开发
---

####一.需求描述
最近在开发一个大数据产品，这个数据产品主要是快速产出用户需要的数据，每次用户调我们数据产品提供的接口后，我们都会把接口的返回值存储到HBase和缓存，存储到HBase目的是后续做一些统计用，存储到缓存的目的加速数据产品的计算速度，因为某些数据在一段时间内计算的结果都是一样的。这个需求可以简单描述成这样，把数据产品的计算结果写缓存和HBase。

<!-- more -->

####二.分析
上述数据产品先计算数据，然后把计算结果存储起来，计算和存储是两个模块，先计算后存储，首先我们想到不应该同步做这两件事情，毕竟存储不存储对使用数据产品的人来说他不关心，他所关心的是我们的数据产品能否准确快速地返回他所需要的数据，这样一旦数据产品计算出结果后，我们应该立即返回给用户，从这点说我们应该在计算模块结束后就把数据直接返回到客户端，因此异步存储计算结果的思路就出来了，不要等结果都存储后，再把计算结果返回，这样可以加速数据产品数据产出的速度，因此我们采用异步存储的思路。
####三.异步存储思路
* 方案1：每次存储都创建一个线程，由这个线程负责把数据写到缓存和HBase
* 方案2：把每次要写的数据先在内存中存储起来，使用固定数目的后台线程把数据写入HBase和缓存

关于方案1是最常见也最容易想到的一种思路，采用固定大小的线程池，每次写数据的时候都new一个线程对象，然后交给线程池去调度执行。

在方案2中我们需要启动N个后台线程，这N个后台线程不断轮循内存中存储数据的BlokingQueue,如果BlockingQueue中有数据，这N个线程会竞争获取一块数据，然后异步把数据写入到HBase和缓存。关于线程的同步我们使用了BlockingQueue，要是BlockingQueue中没有数据，线程就会阻塞，当BlockingQueue中有数据了，线程就会被唤醒。

对于方案2我们需要考虑一下线程的善后处理，当系统中线程池要销毁时，该如何应对呢？此时，我们要保证两个点，第一，保证BlockingQueue中的数据都被存储，第二，保证正在RUNNABLE的线程把数据写完。关于第一点我们可以在线程池销毁的时候，也轮循一下BlockingQueue这个队列，当这个队列中还有数据的话，我们就取数据出来，然后把数据写入HBase和缓存，直到这个队列为空。伪代码如下
```java
while (!blockingQueue.isEmpty()) { // 阻塞队列中还有数据
    // 取出阻塞队列中的数据
    StoreDataObject storeDataObject = blockingQueue.take();
    // 存储数据
    dataWriteService.write(storeDataObject);
}
```

当上面这个代码块执行完后，我们再开始考虑销毁线程池。

接下来我们考虑，如何保证第二点，即保证正在运行的线程把数据写完，我们先让线程池发出一个shutdownNow的信号，此时线程池处于stop状态，拒绝接受新的线程，同时调用线程的interrupt方法，尝试中断线程，要是线程处于RUNNABLE的话，调用interrupt是不会被中断的，由于当BlockingQueue中没有数据的话的，线程就会处于BLOCKED状态，直到BlockingQueue中有数据，线程才会被唤醒，基于这个原理我们就有了下面的思路，伪代码如下
```java
// 先调用线程池的shutdownNow,这样线程池会调线程的interrupt
FIELD_DATA_WRITE_THREAD_POOL.shutdownNow(); 
//处于RUNNABLE的线程不会受影响
while (!FIELD_DATA_WRITE_THREAD_POOL.isTerminated()) {
    // 继续调用线程池的shutdownNow，让线程池调用线程的interrupt
    FIELD_DATA_WRITE_THREAD_POOL.shutdownNow();
}
```

这样我们就保证了上面的第二点，保证处于RUNNABLE的线程能够把数据正常写入到缓存和HBase.

另外，我们写数据的线程都是轮循BlockingQueue的，这些线程需要对中断作出合理响应，即放弃轮循。伪代码如下：
```java
while (true) {
    try {
        StoreDataObject storeDataObject = storeDataQueue.take();
        dataWriteService.writestoreDataObject();
    } catch (InterruptedException e) {
        LOG.info(String.format("%s has bean interrupted!!!", this.getName()), e);
         break; // 轮循终止
    } catch (Exception e) {
        LOG.error(String.format("%s store iquant data exception", this.getName()), e);
        break; // 轮循终止
    }
}
```

至此，方案2表面上看起来是差不多了，但是我们想一下，要是轮循写数据出现异常的时候，线程就退出了，这样消费能力就降低了，这样BlockingQueue中的数据就会堆积，占用很大的内存，从这个角度出发我们需要考虑两个问题，如何保证轮循的线程在挂掉之后，补充一个轮循线程，其次，如何保证BlockingQueue中的数据暴增对系统的不会产生影响，关于第一个问题，我们可以使用一个后台线程去管理这些轮循线程，当轮循的线程数目少于N的时候，这个后台线程自动新增线程，第二个问题有很多解决思路，比如最简单可以限定BlockingQueue的大小，超过大小后，BlockingQueue就拒绝接收数据，把这些被拒绝的数据持久化到某个地方，后续再补充到BlockingQueue中。

####四.最后总结
* 在系统中一旦使用了线程池，在线程池销毁的时候，需要考虑线程正在做什么事情，要不要在销毁之前等线程把自己该做的事情做完，不要贸然就去销毁一个线程池。
* 对于生产者和消费者模型的并发问题，注意考虑如何维持消费能力，以及如何避免共享内存暴增对系统造成的影响。