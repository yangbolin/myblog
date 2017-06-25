title: 关于YGC时间变长的记录
date: 2016-03-19 18:01:23
tags: JVM
categories: 编程开发
---

最近看见很多同事都在讨论一个JVM YGC时间变长的问题，在平时业务开发的过程中，我们经常使用
```java
XStream xs = new XStream();
```
来实现XML和JavaBean之间的相互转换，看看实现就知道在上面的构造函数中不断创建新的classloader出来
```java
public XStream(
            ReflectionProvider reflectionProvider, Mapper mapper, HierarchicalStreamDriver driver) {
        this(reflectionProvider, driver, new ClassLoaderReference(new CompositeClassLoader()), mapper, new DefaultConverterLookup(), null);
    }
```
不断创建新的classloader会导致YGC的时间变长。JVM的类加载机制都是双亲委派机制。
假如：AClassLoader->BClassLoader->CClassLoader，现在需要加载X这个类，AClassLoader首先会交给BClassLoader去加载，BClassLoader会交给CClassLoader去加载，如果CClassLoader能加载到，那么X这个类就被加载了。此时在SystemDictionary这个HashTable数据结构中会存储3条记录。
```java
X-AClassLoader-X.cls
X-BClassLoader-X.cls
X-CClassLoader-X.cls

此时SystemDictionary中有三条关于X的加载记录，如果发现任何一条，就认为X已经加载过了。
```
其中AClassLoader和BClassLoader叫做X的出始类加载器。CClassLoader叫做X的定义类加载器。如果不断自定义ClassLoader的话，SystemDictionary中会不断增加K-V记录，这样YGC扫描的范围就越大，YGC耗时就越多。

最后，在使用XStream时，最好别每次都创建一个新的ClassLoader来，减少YGC的时间，提升性能。