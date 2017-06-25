title: 使用ASM读取class的字节码
date: 2014-07-16 20:18:54
tags: ASM
categories: 编程开发
---

####一.概述
我们平时写的JAVA代码经过编译后就是class文件，class文件其实有非常严谨的结构，具体可以参考[JVM class文件格式规范](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.1),认真读完这个规范，只要你按照这个规范来解析class文件，你就一定能写个javap出来。

<!-- more -->

我们写的JAVA代码最终都是跑在JVM上面的，但是JVM不会跑JAVA源代码，JVM上面跑的是编译后的字节码，也就是说JVM只认识class文件，不认识.java文件。最近在分析Java字节码然后检测一些代码中存在的问题，需要读取Java的class文件，当然按照前面的规范去自己解析class文件也可以，不过这么做你需要处理很多细节问题，ASM就是一个优秀的字节码读取框架，这个框架把byte数组转换成更加具体的对象，方便我们对字节码进行深层次的分析，以及修改，因此最后选择ASM作为字节处理的工具，不过在使用这个工具的时候，发现常量池没法体现出来，因为我只需要常量池中的东西就够了，为了避免全部解析class文件，自己动手解析了常量池。

ASM把class文件定义成下面几部分
![ASM字节码分块](http://bolinyoung.qiniudn.com/ASM-SECTION-PDF.png)

####二.ASM读取Class的字节码文件
使用ASM读取Class的字节码文件时，我们与两种思路可以选择

* Core-API
这中思路其实就是事件的思想，前面我们提到过ASM对字节码进行分块，读取到某一块的时候就会调用相应的visit方法，类似与产生一个事件，然后相应的事件监听者做出相应的响应。这中思路的优点解析过程中内存占用少，速度快，但是缺点就是过了这村就没这店，因为这种解析是顺序的，不可能倒过来的，除非你重新触发一次解析的过程，类似XML的SAX解析思想。流水式的解析，错过了，就永远错过了，除非从头再来。

* Tree-API
Tree-API弥补了Core-API的不足，Tree-API会把解析出来的每一块都保存在内存中，你可以随时获取字节码中的任意一块，这样避免了Core-API的缺点，但是暴漏出来的缺点就是内存占用比较大，类似XML的DOM解析思想。保证字节码的每一块内容在整个解析的过程中一直存储在内存中。

我们可以借鉴ASM这种API设计的理念，解决API设计中存在的一些问题。

![ASM-API核心类图](http://bolinyoung.qiniudn.com/ASM-API.png)

ClassVistor是一个抽象类，我们在使用Core-API的时候需要构造一个自己的Visitor继承ClassVistor即可，然后重载一些ClassVistor中访问各个字节码模块的方法，这样在解析到每个class文件的相应模块时，这些方法会被回调。我们在使用Tree-API的时候，需要构造一个ClassNode,通过上述类图就能看出ClassNode也是ClassVisitor的一个派生类，这不过这个ClassNode在实现这些visit方法的时候把流水式解析的结果存储在一些自己的成员变量中，从而保证各个字节码分块在整个解析的过程中都在内存中一直存在。