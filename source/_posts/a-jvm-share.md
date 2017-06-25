title: 一次JVM分享的相关记录
date: 2014-08-16 15:44:15
tags: JVM
categories: JVM学习
---

####一.概述
最近看见公司一位前辈有分享JVM相关的知识，自己特地从跑过去听了一下，感觉讲的不是很深入，但是相对来说还是不错的，本文就自己听到的一些东西做一个简单的总结。平时我们大多数都是用JAVA写代码,很多人学JAVA都是从HelloWorld开始，但是从你写HelloWorld代码到最终输出HelloWorld到底发生那些事情呢？要想清楚的回答这个问题，并不是很容易，你需要对JVM有一个清晰的了解后，才能说清楚这个问题，这只是一个引子，本文也不去挑战这个问题，这里主要记录一下自己从这个分享中听到的一些感兴趣的东西。

<!-- more -->

####二.JVM运行时数据区
![JVM运行时数据区](http://bolinyoung.qiniudn.com/jvmdataarea.png)

JVM运行时数据区域分为堆内存和非堆内存。注意这里的程序计数器，和CPU没有关系，不要看见Register就认为是CPU的寄存器，每个方法都会由一系列的字节码组成，这里的PC指向的是当前要执行的字节码，执行完字节码后，PC就会向前移动。

####三.类初始化
在JAVA代码中，我们经常写class，JVM加载一个类的时候，先会初始化一个类实例，这个初始化好的类实例存储在perm区中，注意这个类实例不是我们在代码中new出来的对象实例，我们在代码中new出来的对象实例是存储在堆内存中的，也就是说new一个对象的实例和类实例是存储在不同地方的。

![类初始化的过程](http://bolinyoung.qiniudn.com/classinitprocess.png)

类初始化的过程总体上分为Load,Link,Initialize三个阶段，当代码中需要某个类的时候，JVM先会在classpath中尝试去找，如果找不到相应的类，此时会抛出ClassNotFoundException，如果能找到相应的类，就会看这类是否有使用其他的类，有的话，又会去加载当前类依赖的类，接下来会做Link，其实和C/C++编译时的Link是一样的，把当前类所需要的其他类放在一起，看看会不会有问题，Link分为三个阶段，字节码校验，当然这个校验可以通过JVM参数去掉，即verify，prepare阶段会对类的静态成员变量做初始化，比如int会被初始化为0,引用类型会被初始化为null，resolve阶段就是看看当前类所依赖的其他类有没有问题，比如classA依赖classB，在classA中使用classB中的方法f，此时会校验classB中方法f是否存在。最后就是Initialize了，这里的Initialize其实就是执行clinit方法。

注意Link中的Prepare阶段会对类静态成员初始化，我们new一个实例的时候，类的非静态成员变量也有类似初始化的过程，但是这两个初始化不是发生在同一个阶段，new的时候会在堆内存上分配内存空间，此时也会对非静态成员变量做初始化。

下面我们来分析一段代码
```java
public class StaticCode {

    static {
        a = 1;
    }

    private static int  a = 0;
	
    public static void main(String[] args) {
        System.out.println(StaticCode.a);
    }
}
```
这段代码的输出结果是什么？
首先考虑这段代码会不会有编译的问题，提前使用了静态成员变量a。自己编译一下就知道了，这段代码不会有编译的问题。输出结果是0。自己运行一下就知道了。但是为啥结果是0呢？我们来分析一下字节码，看看为啥结果是0
使用javap -verbose来查看字节码，注意字节码中静态代码块
![StaticCode中的静态代码块](http://bolinyoung.qiniudn.com/staticcode-0.png)
其实这个静态代码块对应的方法就是clinit方法。这个方法中有5条字节码指令，偏移量0和1处的字节码表示获取常量1并且赋值给静态成员变量i,这两个字节码指令从哪里来的呢？就是声明静态代码块中对静态成员变量a的赋值操作。偏移量4和5处的字节码表示声明静态成员变量a时的赋值操作，此时这个赋值操作排在静态代码块中赋值操作的后面，因此输出结果为0。

对于上述代码块，如果我们把static代码块的顺序静态成员变量声明的顺序改变一下
```java
public class StaticCode {

	private static int  a = 0;
	
    static {
        a = 1;
    }

    public static void main(String[] args) {
        System.out.println(StaticCode.a);
    }
}
```
此时输出结果为1,因为静态成员变量的声明排在静态代码块的前面，此时静态成员变量声明时的赋值字节码会出现静态代码块中赋值字节码的后面，再看看相关的字节码就能发现
![改变顺序后的字节码](http://bolinyoung.qiniudn.com/staticcode-0-latter.png)

关于静态代码块中的字节码和静态成员变量赋值的字节码谁先执行谁后执行，取决于java代码中静态成员变量和静态代码块出现的先后顺序。

再看一个对象实例化的例子
```java
public class Base {
	private int i = 12;
	public Base() {
		foo();
	}
	
	public void foo() {
		System.out.println(i);
	}
}
public class Child extends Base {
	private int i = 13;
	
	public Child() {
	}
	
	public void foo() {
		System.out.println(i);
	}
	
	public static void main(String[] args) {
		new Child();
	}
}
```
这段代码很简单，在基类构造函数中调用foo方法，这个方法被派生类重写了，此时输出结果为多少？
运行发现此时输出结果为0,首先我们new Child()后在基类构造函数中调用的foo方法一定是在派生类中重写过的foo方法，因为当前实例对象是派生类类型的，初始化实例的时候都是先初始化父类，再初始化子类，这是关键，初始化父类的时候子类还没有初始化好，但是调用子类的foo方法，此时i的值是在分配内存空间时初始化的值，int类型的被初始化为0,因此输出结果为0。我们再来分析一下Child类构造函数的字节码。
![Child类构造函数字节码](http://bolinyoung.qiniudn.com/ChildByteCode.png)
JVM调用每个方法的时候都会给这个方法创建一个方法栈帧，每个方法栈帧都包含一个局部变量表，一个操作数栈，当然还有其他的比如堆栈映射表。局部变量表其实就是一个数组，实例方法中局部变量表的第一元素表示当前实例this，我们看看Child构造函数的字节码发现第一条就是aload\_0，表示把局部变量表的第0个元素放到栈顶，也就是this引用放到栈的顶部，然后执行invokespecial，即调用父类的构造函数，调用结束后，栈顶的this引用会被弹出，因此需要继续给栈顶压一个this引用，因为后面要对成员变量i赋值，执行aload\_0后就能把this引用再次压入到栈顶部，bipush 13标识把13转换成int型的值，然后压入到栈的顶部，putfield表示弹出栈顶的整型值，赋值给成员变量i，此时栈顶部就是this引用了，this引用也会被弹出，因为赋值的时候需要知道变量i是哪个实例的成员变量。分析完这段字节码后，输出结果为0也就很好解释了。因为调用基类构造函数时，派生类成员变量i的赋值还没有进行。

JVM有一个限制，不允许多个线程去初始化同一个类，当一个线程初始化ClassA的时候，另外一个线程要是也来初始化ClassA的话，此时只能等待，等迁移个线程初始化完后后一个线程直接返回。下面这例子就是来证明这个结论
```java
class Lock {}

class Danger {
  static {
    System.out.println("clinit begin...");
    try {
    	Thread.sleep(2000);
    } catch (Exception e) {
    	System.out.println(e);
    }
    synchronized (Lock.class) 
    { 
    	System.out.println("clinit done!");
    }
  }
}

public class Test {
  public static void main(String[] args) {
  	// 创建一个线程,此处标记为Thread1，方便下面好描述
    new Thread() {
      public void run() {
        synchronized (Lock.class) {
          System.out.println("new thread start!");
          try {
          	Thread.sleep(1000);
          } catch (Exception e) {
          	System.out.println(e);
          }
          new Danger();
        }
        System.out.println("new thread end!");
      }
    }.start();
    try {
    	Thread.sleep(500);
    } catch (Exception e) {
    	System.out.println(e);
    }
    
    System.out.println(new Danger());
    System.out.println("DONE!");
  }
}
```

上述代码运行就会死锁。
现在我们来分析为什么会死锁，线程Thread1先执行获取到Lock.class上的锁，然后sleep等待1000ms,此时Main线程sleep等待500ms，Main线程睡醒后，开始创建Danager实例，然后在创建Dananger实例时执行Danager类实例初始化的代码，在类初始化的代码中sleep等待2000ms,在这个过程中Thread1睡醒了，然后也去创建Danger实例，当然也需要执行类实例初始化的代码了，但是Main线程正在执行Danger类实例的初始化，因此Thread1只能等待，等Main线程睡醒了，但是需要等Thread1释放Lock.class上的锁才能完成Danger类实例初始化，此时Thread1等Main线程初始化完Danger类，Main线程等Thread1释放Lock.class上的锁，因此Thread1和Main互等，形成死锁，死锁的本质原因就是JVM不允许多个线程同时初始化一个类实例。

同事F说这段代码写的太复杂了，不太好理解，的确，然后F给出了一个更简单的例子证明JVM不允许多个线程同时初始化一个类实例。
```java
public class ShutDownDeadLock {

    public static final Object referObject = new Object();
    static {
        Runtime.getRuntime().addShutdownHook(new Thread() {

            @Override
            public void run() {
                if (referObject == null)
                ;
            }

        });
        System.exit(0);
    }

    public static void main(String[] args) {

    }
}
```
上述代码会导致JVM没法退出，static代码块中在类初始化的时候会执行，System.exit(0);会导致JVM退出，但是有一个JVM退出时回调的钩子线程，这个线程会访问ShutDownDeadLock的静态成员变量referObject，但是此时ShutDownDeadLock类实例不允许其他线程使用，因为当前类实例正在被初始化，被JVM加锁了。

上面的代码做如下修改就能正常退出了
```java
public class ShutDownUnDeadLock {
    public static final Object referObject = new Object();
    
    static {
        final Object oo = referObject;
        
        Runtime.getRuntime().addShutdownHook(new Thread() {

            @Override
            public void run() {
                if (oo == null)
                ;
            }

        });
        System.exit(0);
    }
    

    public static void main(String[] args) {

    }
}
```
此时JVM能够正常退出，因此线此时访问oo时不需要通过ShutDownUnDeadLock类实例来访问，直接从方法栈帧的局部变量表中读取，也就是说直接从栈上读取。

我们经常在代码中定义常量
```java
public class IQuantConstant {
	public static final int IQUANT_PAGE_SIZE = 30;
}
```
```java
public class IQuantConstantTest {
	public static void main(String[] args) {
		System.out.println(IQuantConstant.IQUANT_PAGE_SIZE);
	}
}
```
如果IQuantConstant和IQuantConstantTest在不同的JAR包中，此时修改了IQuantConstant中常量IQUANT_PAGE_SIZE的值后，另外一个JAR包感知不到，这个是编译器做的优化，编译的时候直接把IQUANT_PAGE_SIZE的值拷贝过去。

TLAB是指thread local allocation buffers，用来避免线程竞争同步的开销。

一个class能被卸载的前提是加载这个class的classloader加载的所有类都能被卸载，这个在使用groovy脚本时要注意，容易引起classloader的泄漏，导致perm区OOM。perm区的gc就能卸载满足条件的class。

####三.GC相关
![CMS](http://bolinyoung.qiniudn.com/cmsgc.png)
使用CMS垃圾回收的时候，会有两次stop-the-world,因此使用jstat -gcutil查看FGC次数时，发现每次FGC都是增加2。

新生代分为eden区，s0和s1,之所以有s0和s1两个对等的区，是因为MinorGC采用的是拷贝复制的算法进行垃圾回收，MinorGC会把存活的对象从s0拷贝到s1，或者从s1拷贝到s0。

GC算法
标记-清除，该算法先去标记可回收的对象，然后再一个个回收调，容易产生内存碎片。

复制，该算法把内存划分为两个对等的区域，把存活的对象直接从一个区域拷贝到另外一个区域。

标记-整理，先标记出需要可回收的对象，然后对不可回收的对象进行移动，让不可回收的对象集中在内存的一端。