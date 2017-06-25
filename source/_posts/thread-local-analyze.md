title: JAVA线程本地存储之ThreadLocal的分析
date: 2014-10-07 21:01:03
tags: JAVA 
categories: 编程开发
---

#### 一.概述
ThreadLocal是JDK的一个线程本地存储的类，我们可以把一些线程私有的数据写在ThreadLocal中，这样这些数据只有一个线程可见，实现了所谓的栈封闭。这样存储一些线程私有的数据，我们就不用去费心考虑如何保证临界资源的互斥访问了，同时对于一个线程，这些私有数据也只做一次初始化。

<!-- more -->

#### 二.一段ThreadLocal的测试代码
```java
class LocalObject {
    private String name;
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}

class LocalStoreThread extends Thread {
    /** 定义了一个线程本地存储的成员变量 **/
    private ThreadLocal<LocalObject> threadLocal = new ThreadLocal<LocalObject>();

    public LocalStoreThread(LocalObject lo) {
        threadLocal.set(lo);
    }

    @Override
    public void run() {
        System.out.println(threadLocal.get().getName());
    }
}

/**
 * <pre>
 * 创建一个线程实例，这个线程实例中有一个线程本地存储成员变量
 * </pre>
 */
public class ThreadLocalTest {
    public static void main(String[] args) {
        LocalObject lo = new LocalObject();
        lo.setName("thread-local");
        new LocalStoreThread(lo).start();
    }
}
```
上述代码运行的时候在run方法中抛出了空指针异常，明明在构造函数中调用了threadLocal的set方法，为什么get的时候获取到了null,然后使用了null抛出了NPE呢？

由于ThreadLocal是和线程相关的，我们上面的代码在够在函数中往线程本地存储变量中设置了一个实例对象，在run方法中获取这个实例对象的时候发现拿到是null,所以我们有必要看一下set时对应的线程和get时对应的线程是不是一样的。因此在set之前打印一下Thread.currentThread()，同时在get之前打印一下Thread.currentThread()

```java
class LocalObject {
    private String name;
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}

class LocalStoreThread extends Thread {
    /** 定义了一个线程本地存储的成员变量 **/
    private ThreadLocal<LocalObject> threadLocal = new ThreadLocal<LocalObject>();

    public LocalStoreThread(LocalObject lo) {
        // set之前打印当前线程
        System.out.println(Thread.currentThread().getName());   // main
        threadLocal.set(lo);
    }

    @Override
    public void run() {
        // get之前打印当前线程
        System.out.println(Thread.currentThread().getName()); // Thread-0
        System.out.println(threadLocal.get().getName());
    }
}

/**
 * <pre>
 * 创建一个线程实例，这个线程实例中有一个线程本地存储成员变量
 * </pre>
 */
public class ThreadLocalTest {
    public static void main(String[] args) {
        LocalObject lo = new LocalObject();
        lo.setName("thread-local");
        new LocalStoreThread(lo).start();
    }
}
```
好，问题出现了，set时的当前线程和get时的当前线程不一样，所以get的结果是null。set是写在线程的构造函数中的，此时当前线程是main线程，因为在main线程中创建线程。但是在run方法中当前线程已经不是main线程了变成了new出来的这个新线程了。

####三.ThreadLocal中get和set方法分析
```java
public T get() {
        // 获取当前线程实例
        Thread t = Thread.currentThread();
        /* 获取当前线程实例的ThreadLocalMap，其实就是一个数组
         * 这个数组可以扩容，每次空间不够时都拿当前大小*2
         */
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            /*根据this哈希获取数组中的一个元素*/
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        // 如果当前线程的ThreadLocalMap为null,就创建一个
        return setInitialValue();
}
private T setInitialValue() {
        /* 这里调用了ThreadLocal的initValue方法，一般都会在创建ThreadLocal
         * 实例的时候重写这个方法，比如说ThreadLocal中要是存放数据库链接对象的
         * 话，就需要一个初始化方法来初始化这个数据库链接对象
         */
        T value = initialValue();
        /*把初始化好的值保存起来*/
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
}
/*创建线程的ThreadLocalMap*/
void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
}
/*获取线程的ThreadLocalMap*/
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
}
```
上述代码就是ThreadLocal的get源代码，先根据当前线程获取当前线程的ThreadLocalMap,此时获取到的就是一个table数组，接下来根据ThreadLocal实例的threadLocalHashCode来获取table数组中的一个元素，这个元素是个K-V的键值对，此时V就是本地存储的值。

```java
/*关于set代码和get代码是对称的*/
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
}
```

####四.ThreadLocal源代码中提供的一个实例代码
```java
import java.util.concurrent.atomic.AtomicInteger;

public class UniqueThreadIdGenerator {
    private static final AtomicInteger uniqueId = new AtomicInteger(0);  

    private static final ThreadLocal < Integer > uniqueNum =   
            new ThreadLocal < Integer > () {  
        //定义初始值（副本）  
        @Override protected Integer initialValue() {  
            return uniqueId.getAndIncrement();  
        }  
    };  

    public static int getCurrentThreadId() {  
        // 这里应该要把 uniqueId换成uniqueNum，源码应该是写错了   
        return uniqueNum.get();  
    }

    static class MyThread extends Thread {

        @Override
        public void run() {
            System.out.println(UniqueThreadIdGenerator.getCurrentThreadId());
        }
    }
    public static void main(String[] args) {
        for (int i = 0; i < 5; ++i) {
            new MyThread().start();
        }
    }
}
```
这里总共创建了5个线程，每个线程在run方法中调用UniqueThreadIdGenerator.getCurrentThreadId()时，发现每个线程的ThreadLocalMap都是null,所以每次初始化的方法initialValue都会被调用。

####五.Thread&&ThreadLocal&&ThreadLocalMap之间的关系
![图示](http://dl.iteye.com/upload/attachment/0084/5636/079ae373-cc30-3a5e-8890-496018582ca0.bmp)

####六.总结
一个线程只有一个ThreadLocalMap,其实ThreadLocalMap就是一个table数组，数组中的每个元素都是一个K-V的键值对，其中K就是ThreadLocal实例，在获取本地存储的值的时候根据ThreadLocal实例的threadLocalHashCode来对table进行Hash查找，找到对应的K-V键值对。一个线程可以有多个ThreadLocal实例，那么有多少个ThreadLocal实例就决定了table数组的大小，这个数组是动态增长的，每次要是大小不够，就自动扩充为原来大小的2倍，然后对于原来的元素重新Hash,这个操作的成本还是很大的。

