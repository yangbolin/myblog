title: javac生成的access$000方法
date: 2015-03-21 10:07:34
tags: JAVA
categories: 编程语言
---
####一.概述
最近在做字节码分析的时候，发现字节码中出现了access$000的方法，这个方法不是开发工程师写的，是编译器生成的，那么编译器生成这个方法是为了解决什么问题呢？我们知道编译会为静态代码块生成执行的方法，在类没有构造函数时为类生成缺省构造函数，这个access$000究竟是在什么场景下面用的呢？仔细分析包含access$000的类，发现该类中存在内部类，结合google搜索，原来acess$000是为了解决内部类访问外部类的成员，包括成员变量和成员方法。

<!-- more -->

####二.深入分析
我们先写个简单的测试例子，一个类中包含一个内部类
```java
public class Outer {
    private int x = 3;
    private void f() {
        
    }
    
    class Inner {

        public void f() {
            Outer.this.f();
            System.out.println(Outer.this.x);
        }
    }
    
    public static void  main(String[] args) {
        System.out.println("Inner Test");
    }
}
```
然后我们先看内部类的字节码
![内部类字节码](http://bolinyoung.qiniudn.com/inner-call.png)
我们看到内部类的字节码中调用了access$000和access$100这两个方法，这两个方法都是Outer这个类的静态成员方法，同时带有一个Outer类型的参数。那么我们看看Outer这个类的字节码到底是什么样子
![外部类的字节码](http://bolinyoung.qiniudn.com/outer-code.png)
果然外部类中存在两个这样的方法，我们先看一下access$000这个方法的实现，该方法先用aload_0把方法参数入栈，然后调用栈顶元素的f方法，即Outer的f方法，因为内部类中有地方通过外部类的this引用调用Outer的f()方法。access$100这个方法，很明显在访问Outer的成员变量x。

至此疑问都清楚了，原来access$xxx是编译器生成的，用来解决内部类访问外部类的成员。

但是你一定有一个疑问，我们自己能否写access$xxx的方法呢？答案是可以的。

####三.自己编写access$xxx
写个简单的例子
```java
public class Outer {
    private int x = 3;
    private void f() {
        
    }
    
    class Inner {

        public void f() {
            Outer.this.f();
            System.out.println(Outer.this.x);
        }
    }
    
    public static void  main(String[] args) {
        System.out.println("Inner Test");
    }
    
    // 编写自己的access$xxx方法
    static int access$888(Outer outer) {
    	return outer.x;
    }
}
```

> 注意:

> * 我们自己编写的access$xxx不能和编译生成的具有相同的方法签名，否则你会得到一个编译错误的。
> *  access$xxx可以绕过编译器的检查，访问类的私有成员。
> * 一般不要编写access$xxx方法。

