title: NoClassDefFoundError VS ClassNotFoundException
date: 2014-05-31 23:52:45
tags: JAVA
categories: 编程开发
---

在开发JAVA项目的时候，我们经常会遇到这样两种异常，一种是java.lang.NoClassDefFoundError,另外一种是java.lang.ClassNotFoundException。

<!-- more -->

对于ClassNotFoundException，意思说是需要加载的类压根在classpath中不存在，我们需要使用一个类的时候，要是类加载器没有加载过这个类，classloader就会在classpath中寻找这个类，然后使用合适的类加载器去加载这个类，要是找不到相应的类文件就会抛ClassNotFoundException异常。出现这个异常一般都是少引入了jar或者jar包冲突导致。

对于NoClassDefFoundError，不是说在classpath中找不到相应的类文件，而是找到了相应的类文件，但是实例化这个类的时候出错了，JVM加载的每个类都会对应一个java.lang.Class类型的实例。此时我们需要检查的是类的static代码块或者static的成员变量，很有可能就是这些static的东东在初始化的发生了问题。例如

```java
class xxx {
    // 静态成员变量也会导致这个问题，比如初始化log的时候可能会出现jar包冲突的问题,这也会抛出NoClassDefFoundError的异常
    private static Logger LOG = Logger.getLogger("xxx");
    
    static {
        // 加载xxx的时候先会执行这个static代码块，要是这个static代码块执行有问题，那么就会出现NoClassDefFoundError的异常
    }
}
```

出现这个异常的时候，重点关注static相关的代码就会找到问题所在。

下面我们看看java语言规范中关于这两个异常的定义

> java.lang.ClassNotFoundException
> Thrown when an application tries to load in a class through its string name using: 
> - The forName method in class Class. 
> - The findSystemClass method in class ClassLoader . 
> - The loadClass method in class ClassLoader. 

> but no definition for the class with the specified name could be found. 

> java.lang.NoClassDefFoundError
> Thrown if the Java Virtual Machine or a ClassLoader instance tries to load in the definition of a class (as part of a normal method call or as part of creating a new instance using the new expression) and no definition of the class could be found. 
> The searched-for class definition existed when the currently executing class was compiled, but the definition can no longer be found. 

不翻译了，很简单的英文，看英文更好理解^=^,关于码代码，一定要学会看英文文档，要是什么事情都等中文，那就迟了。