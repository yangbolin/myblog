title: Java 函数参数引用思考
date: 2014-04-08 22:05:56
tags: JAVA
categories: 编程语言
---

###一.概述
Java是一门纯粹的面向对象语言，除了8种基本类型意外，其他都是对象。函数是编程语言中的一个非常重要的概念，Java中的函数都不是孤立存在的，不像C或者JS你可以定义一个孤立的函数，有函数就有函数调用，在Java中函数参数就两种类型，要么函数参数是基本类型，要么函数参数是引用类型。函数调用就会涉及到的参数的传递，在Java中，参数传递就一种方式，既所谓值传递，其实就是传递变量值的拷贝，如果参数类型是基本类型，那么传递的就是基本类型参数值的一份拷贝，如果参数类型是引用类型，那么传递的就是引用值的一份拷贝，而不是引用对象的一份拷贝，其实用C语言的指针来说，传递的就是指针所指内存空间地址的一份拷贝，正所谓Java中虽然没有指针，但全是指针。

<!-- more -->

###二.代码实例
```java
public class ParamterReference {
    // 注意这段代码
    public static void test(Object res) {
        	res = new Object();
        	// 这个res指向test方法中new出来的Object
        	System.out.println(res);
    }
    public static void main(String[] args) {
        	Object res = new Object();
        	// 此时此刻res指向main方法中new出来的Object
        	System.out.println(res);
        	test(res);
        	// 此时此刻res指向main方法中new出来的Object
        	System.out.println(res);
    }
}
```
上面代码运行的结果如下:


java.lang.Object@2e6e1408
java.lang.Object@3ce53108
java.lang.Object@2e6e1408


我们从结果中发现main函数中两个地方打印出来的的res是一样，但是和test方法中打印的结果不一样，这说明res在main函数和test方法中指向的不是同一个java实例，用C语言的话来说，就是指向不同的内存空间。

在调用test方法的时候,main函数中的res引用值被覆盖了一份保存到test函数栈帧上,此时此刻有两个函数栈帧，一个main函数的栈帧，一个test函数的栈帧，main函数中res的引用值也有两份，一个存在main函数栈帧上，一个存在test函数栈帧上，但是这两个函数对应栈帧上的值是一样的，只是由不同的变量所持有，再套用C语言中的一句话，此时此刻，就是两个不同的指针变量指向了同一块内存区域。

因此，我们在test内部修改了test函数栈帧上的一个局部变量引用的值，这个局部变量在栈帧销毁之后，生命周期就结束了，因此我们在函数内部对这个局部变量值的修改不会被外部调用函数感知到。当然要是我们利用引用修改了引用所指内存空间的值，外部调用函数一定会感知到。
###三.最后总结
* 不要在函数内部修改引用类型参数的引用值，这个修改对外部调用者来说完全感知不到，出现这样的代码，很可能就是一个BUG。
* 通过引用对象对引用对象所指的内存空间进行写入，这个操作一定会被外部调用者感知到。
