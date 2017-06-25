title: Java局部变量有没有final关键字对JVM性能影响不大
date: 2014-05-17 18:08:12
tags: [JAVA,JVM]
categories: 编程语言
---

####一.概述
在写Java代码中我们经常使用final这个关键字，被final修饰的变量，编译器不允许我们修改它的值，如果是基本类型，编译器不允许我们在第一次赋值后再修改基本类型的值，如果是引用类型，编译器不允许我们在第一次赋值后再修改引用的指向，这都是编译器层面的限制，因为如果你试图去修改它，就会得到一个编译错误。

<!-- more -->

####二.如何证明final关键字对JVM的性能影响不大
我们写下面的Java代码
```java
public class FinalLocalVariableTest {
	public int generateInt() {
		return 3;
	}
	
	public int f1() {
		final int a = generateInt();
		final int b = generateInt();
		return a + b;
	}
	
	public int f2() {
		int a = generateInt();
		int b = generateInt();
		return a + b;
	}
	
	public int f3() {
		final int a = 2;
		final int b = 3;
		return a + b;
	}
	
	public int f4() {
		int a = 2;
		int b = 3;
		return a + b;
	}
}
```

上面的java代码很简单，我们分组来对比，首先看看f1和f2的写法，f1和f2的写法一样，只是在f1是使用了final类型的局部变量，但是在f2中没有使用final类型的局部变量，此时我们编译上面的代码后，使用javap -verbose来查看编译后f1和f2的字节码
![函数f1的字节码](http://bolinyoung.qiniudn.com/f1.png)
红色框内的字节码表示局部变量量a,b的赋值
![函数f2的字节码](http://bolinyoung.qiniudn.com/f2.png)
红四框内的字节码表示final类型局部变量a,b的赋值

对比发现，使用final关键字声明局部变量和不使用final关键子声明局部变量，生成的字节码都是一样的，对于JVM来说，最终执行的是字节码，既然字节码都一样，那么执行的性能也一样。

接下来我们看看f3和f4的字节码有何不同
![f3&&f4的字节码](http://bolinyoung.qiniudn.com/f3&&f4.png)
观察上图，我们发现f3中使用了final关键字声明了局部变量a和b，此时a,b的值在编译阶段就能确定，因此字节码中直接返回常量5,但是f4中我们没有使用final关键字，因此先要把变量值load到方法栈上，然后执行变量的相加操作，最后返回相加的结果，不过这个对JVM的性能影响较小。

综上所述：
使用不使用final关键字修饰局部变量，对JVM的性能影响不大。
