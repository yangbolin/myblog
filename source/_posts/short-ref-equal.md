title: Short对象的比较
date: 2014-12-26 23:02:20
tags: JAVA
categories: 编程开发
---
####一.背景
今天和同事一起分析线下问题，发现代码中有个笔误，对两个Short类型的对象使用==来比较，结果是比较表达式一直为false，导致一个bug的产生。

<!-- more -->

####二.分析
```java
public class Test {

    public static void main(String[] args) {
        Short s = 100;
        Short s1 = 100;
        // true
        System.out.println(s1 == s);

        Short t = 129;
        Short t1 = 129;
        // false
        System.out.println(t1 == t);
        
        Short m = new Short("100");
        Short m1 = new Short("100");
        // false
        System.out.println(m1 == m);
    }
}
```
写了一小段代码，模拟今天遇到的问题。我们看到第一个输出为true，第二个输出为false，第三个输出为false。为什么呢？我们来看一下生成的字节码。

```java
Code:
   // Stack表示操作数栈的最大深度
   // Locals表示局部变量表所需的存储空间，单位是Slot，JVM为局部变量分配内存所使用的最小单位，double和long占用了两个Slot
   // Args_size 表示方法参数个数，main函数只有一个参数，当然size为1了
   // 明明main方法中只有6个局部变量，为啥空间大小是7,别忘了第一个存储的是调用方法的实例，如果是静态方法存储的是类实例，如果是非静态方法存储的是this引用。
   Stack=3, Locals=7, Args_size=1

   0:	bipush	100
   // 这里调用了Short类的静态方法valueOf,待会分析valueOf的实现
   2:	invokestatic	#16; //Method java/lang/Short.valueOf:(S)Ljava/lang/Short;
   5:	astore_1
   6:	bipush	100
   8:	invokestatic	#16; //Method java/lang/Short.valueOf:(S)Ljava/lang/Short;
   11:	astore_2
   12:	getstatic	#22; //Field java/lang/System.out:Ljava/io/PrintStream;
   15:	aload_2
   16:	aload_1
   17:	if_acmpne	24
   20:	iconst_1
   21:	goto	25
   24:	iconst_0
   25:	invokevirtual	#28; //Method java/io/PrintStream.println:(Z)V
   28:	sipush	129
   31:	invokestatic	#16; //Method java/lang/Short.valueOf:(S)Ljava/lang/Short;
   34:	astore_3
   35:	sipush	129
   38:	invokestatic	#16; //Method java/lang/Short.valueOf:(S)Ljava/lang/Short;
   41:	astore	4
   43:	getstatic	#22; //Field java/lang/System.out:Ljava/io/PrintStream;
   46:	aload	4
   48:	aload_3
   49:	if_acmpne	56
   52:	iconst_1
   53:	goto	57
   56:	iconst_0
   57:	invokevirtual	#28; //Method java/io/PrintStream.println:(Z)V
   60:	new	#17; //class java/lang/Short
   63:	dup
   64:	ldc	#34; //String 100
   // 这里直接调用了Short的构造函数
   66:	invokespecial	#36; //Method java/lang/Short."<init>":(Ljava/lang/String;)V
   69:	astore	5
   71:	new	#17; //class java/lang/Short
   74:	dup
   75:	ldc	#34; //String 100
   // 这里直接调用了Short的构造函数
   77:	invokespecial	#36; //Method java/lang/Short."<init>":(Ljava/lang/String;)V
   80:	astore	6
   82:	getstatic	#22; //Field java/lang/System.out:Ljava/io/PrintStream;
   85:	aload	6
   87:	aload	5
   89:	if_acmpne	96
   92:	iconst_1
   93:	goto	97
   96:	iconst_0
   97:	invokevirtual	#28; //Method java/io/PrintStream.println:(Z)V
   100:	return
```

我们在字节码中看到有调用Short的valueOf方法，那么接下来我们看看valueOf方法的实现。

```java
...
private static class ShortCache {
	private ShortCache(){}

	static final Short cache[] = new Short[-(-128) + 127 + 1];

	static {
	    // 缓存[-127,128]之间的Short对象
	    for(int i = 0; i < cache.length; i++)
		cache[i] = new Short((short)(i - 128));
	}
}
...
public static Short valueOf(short s) {
	final int offset = 128;
	int sAsInt = s;
	// 走缓存
	if (sAsInt >= -128 && sAsInt <= 127) { // must cache 
	    return ShortCache.cache[sAsInt + offset];
	}
        return new Short(s);
}
...
```
看完valueOf的实现，你就明白这一切了，首先使用=赋值的时候会调用valueOf(S)这个方法，S标识short，在这方法中如果要赋予的值在[-127,128]这个区间内，那么直接取缓存中的值，缓存中缓存了[-127,128]这个区间内的所有Short对象，如果Short的值在这个区间内，你使用=赋值，直接取缓存，至此上面的问题就不言而喻了。

####三.总结
> 1.写代码的时候最好不要使用==来比较任何类型的引用，除非是基本类型，不然很容易出问题，导致某些场景下面没问题，某些场景下面有问题。
> 2.Long类型也有类似的机制。
