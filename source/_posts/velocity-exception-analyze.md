title: 一次velocity异常的分析
date: 2014-08-22 22:05:14
tags: velocity
categories: 编程开发
---

####一.现象
最近在使用velocity模板的时候RuntimeInstance的1103行代码出现了NPE,如下图所示
![velocity-npe](http://bolinyoung.qiniudn.com/velocity-npe.png)
该行代码一直在抛NPE。

<!-- more -->

####二.分析
RuntimeInstance的1103行出现NPE原因只有一个，那就是parserPool是NULL，现在我们只要搞清楚parserPool为啥在这里是NULL，我们就知道这个NPE出现的原因了，看上面的图，我们发现在执行1103行代码的时候调用了requireInitialization()，那么此时需要看看requireInitialization()到底做了那些事情。
![requireInitialization](http://bolinyoung.qiniudn.com/requireInitialization.png)
我们看到了requireInitialization中调用了init方法，但是这个方法可能抛出异常。关于requireInitialization方法中的代码很有疑问，首先想到的是并发问题，这里怎么没考虑到多个线程并发初始化的问题呢？其次，initialized标识当前实例是否已经被初始化了，能进入到if分支中，说明当前实例肯定没有被初始化，因此initialized值一定是false，initializing标识当前实例正在被初始化,能进入到if分支中，说明当前实例没有被正在初始化，因此initializing的值一定是false。接下来我们看看init的代码
![init](http://bolinyoung.qiniudn.com/init.png)
一开始怀疑并发的问题，看了init的代码后，这个点就不用再去怀疑了。init中也有一个if分支，进入if分支的逻辑和requireInitialization进入if分支的逻辑是一样的，这里就不再进行分析了，关键看2处的代码，此时会标记当前实例正在被初始化，加入initializeParserPool()方法中抛出了一个Exception，此时下面的代码就不会被执行，当前方法栈帧直接退出，当前线程获取的锁也会被释放(后面给出验证的例子)，此时initializing值没有机会设置成false了，也就是说3处的代码没法机会执行了，直接回到requireInitialization方法中，该方法中捕获了这个异常，此时问题就出现了，等下次再初始化的时候requireInitialization中的init方法永远就不会被调用了，此时1103行代码就一直NPE了，由于错过了第一次init时的真正异常，后面看到就一直是NPE，不知道NPE的具体原因是啥了，也就是说requireInitialization中catch住的异常后续发现不了。

####三.解决思路
通过上面的分析，我们只要找到requireInitialization中catch住的异常是啥，问题也就差不多解决了，因为第一次初始化的时候异常已经发生了，后续的初始化都不会进入到requireInitialization中的if分支了，此时要么重启系统去看第一初始化时异常是什么，要么找日志，debug神器中可以动态修改变量的值，那么在requireInitialization的if分支处设置断点，然后动态修改initializing的值为false,此时异常出现了

```
Failed to initialize an instance of org.apache.velocity.runtime.log.Log4JLogChute with the current runtime configuration.
```
 异常信息表明可能log有问题，因为出现Log4J关键词了，这个异常很陌生，不知如何解决，直接去谷歌一下就能解决问题，一般参考stackoverflow上的答案，有人说在velocity引擎初始化的时候增加下面代码即可

```java
Properties p = new Properties();
p.setProperty("runtime.log.logsystem.class", "org.apache.velocity.runtime.log.NullLogSystem");
try {
      INSTANCE.init(p);
} catch (Exception e) {
      LOG.error(e);
}
```
索性试一下，发现问题解决啦。

####四.总结
1.个人认为RuntimeInstance的requireInitialization方法有BUG，应该增加finally分支，在该分支中把initializing设置为false，后续可以考虑修复一下velocity的这个BUG，在开源项目中贡献一些代码。
2.遇见陌生异常时记得谷歌，记得stackoverflow。
3.在分析的过程中，自己突然想到一个问题，synchronized修饰的方法内部抛出Exception后当前线程获取到锁会不会释放，答案是会释放，深层次的分析估计要看JVM源码了，这里给出验证的例子
```java
public class SynchronizedTest {

    public synchronized void init() throws Exception {
        if (true) {
            throw new Exception("test exception");
        }
    }
    
    public static void main(String[] args) {
        SynchronizedTest oo = new SynchronizedTest();
        
        new ThreadModel1(oo).start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new ThreadModel2(oo).start();
    }
}

class ThreadModel1 extends Thread {

    private SynchronizedTest synchronizedTest;
    public ThreadModel1(SynchronizedTest synchronizedTest) {
        this.synchronizedTest = synchronizedTest;
    }
    
    @Override
    public void run() {
        try {
            synchronizedTest.init();
        } catch (Exception e) {
            System.out.println("ThreadModel1 Exception...");
            while(true) {}
        }
    }
}

class ThreadModel2 extends Thread {

    private SynchronizedTest synchronizedTest;
    public ThreadModel2(SynchronizedTest synchronizedTest) {
        this.synchronizedTest = synchronizedTest;
    }
    
    @Override
    public void run() {
        try {
            synchronizedTest.init();
        } catch (Exception e) {
            System.out.println("ThreadModel2 Exception...");
        }
    }
}
```

输出结果:
ThreadModel1 Exception...
ThreadModel2 Exception...

ThreadModel2线程能继续运行并且输出结果，说明init上的锁已经被ThreadModel1线程所释放。