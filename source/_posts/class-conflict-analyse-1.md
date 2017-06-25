title: 一个jar包冲突的解决
date: 2014-05-09 18:59:01
tags: JAVA
categories: 编程开发
---

最近在发布一个需求的时候，遇到下面的错误

<!-- more -->

![出错截图](http://bolinyoung.qiniudn.com/conflict1.png)

应用主干部署出现这个问题，从日志中看貌似是jar包冲突导致，上图中红色框圈起来的内容是关键信息

> java.lang.NoSuchMethodError: org.objectweb.asm.ClassWriter.<init>(Z)V

找不到ClassWriter类的构造函数，即这行出错日志所要表达的信息。

<init>(Z)V是构造函数ClassWriter(boolean arg0)的字节码，其中V标识函数返回值void。这时候错误很明显了，就是说JVM自己加载的ClassWriter类没有ClassWriter(boolean arg0)这样的构造函数，但是系统需要一个这样的构造函数，至此，问题很明确，jar包冲突啦。但是到底是那个jar包冲突了呢？

使用mvn dependency:tree打了依赖树，发现org.objectweb.asm.ClassWriter这个类在一个asm的jar包中，但是asm这个jar包我们的应用没有显式依赖，都是间接依赖，难道要一个一个去仲裁掉吗？

此时，我没有继续排查，而是把上面那行java.lang.NoSuchMethodError: org.objectweb.asm.ClassWriter.<init>(Z)V作为关键词到google搜了一把，很快就有答案说升级sourceforge.cglib到nodep-2.2，因此在顶级pom中增加了
```xml
<dependency>
    <groupId>com.alibaba.external</groupId>
    <artifactId>sourceforge.cglib</artifactId>
    <version>nodep-2.2</version>
</dependency>
```
同时在相应的子pom中也配置相关的依赖，重启应用，问题解决。

关于jar包冲突经常会说某个类的方法找不到，但是具体是什么方法，显示出来的都是字节码，因此我们需要了解一些字节码的知识，才能看到具体是那个方法。其中关于类型相关的可以看一下这个联合体的定义
```c
typedef union jvalue { 
    jboolean z; 
    jbyte    b; 
    jchar    c; 
    jshort   s; 
    jint     i; 
    jlong    j; 
    jfloat   f; 
    jdouble  d; 
    jobject  l; 
} jvalue; 
```

下面给出具体类型对照

|Type Signature|Java Type|
|---|---|
|Z|boolean|
|B|byte|
|C|char|
|S|short|
|I|int|
|J|long|
|F|float|
|D|double|
|L fully-qualified-class ;|fully-qualified-class|
|[ type|type[]|
|( arg-types ) ret-type|method type|

例如
```java
long f (int n, String s, int[] arr); 
```
具有如下的字节码签名
> (ILjava/lang/String;[I)J 