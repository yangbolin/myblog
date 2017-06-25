title: 接口中使用泛型参数的思考
date: 2014-05-27 20:43:57
tags: JAVA
categories: 编程语言
---

今天在使用一个别人提供的接口时，发现该接口的定义如下：
```
public interface xxxService<T extends BasicModel> {
    public T getXXX();
    public T countXXX();
}
```

<!-- more -->

看到上面的接口，我们发现这个接口需要一个泛型参数，同时这个泛型参数必须是BasicModel或者BasicModel的派生类型。同时我们也应该想到xxxService可能会有多种实现，因为不同的泛型参数对应不同的实现。比如
```java
public class xxAServiceImpl implements xxxService<A extends BasicModel> {
    public A getXXX() {
        A a = xxx;
        ...
        return a;
    }
    public A countXXX() {
        A a = xxx;
        ...
        return a;
    }
}
```

```java
public class xxBServiceImpl implements xxxService<B extends BasicModel> {
    public B getXXX() {
        B b = xxx;
        ...
        return b;
    }
    public B countXXX() {
        B b = xxx;
        ...
        return b;
    }
}
```

这样可以提供不同粒度的服务化接口的实现，但是有一个统一定义的地方，可以说这个泛型的接口是一个高度的抽象。

此时你可能会考虑到Spring的注入，针对这种泛型接口，Spring的注入也没有啥问题，基于上面的实现，我们可以写下面的spring配置文件
```xml
	<bean id="xxAService" class="xx.xx.xxAServiceImpl" />
	<bean id="xxBService" class="xx.xx.xxBServiceImpl" />
```
此时我们要是在其他类中按照名称注入这两个bean
```java
public class xxxService {
    private xxxService<A> xxAService;
    private xxxService<B> xxBService;
    // 这里省略setter方法，或者你可以使用@Resource(name="xxx")来按照名称注入
}
```
这样注入是没有任何问题的。
但是要是想按照类型注入，那就出现注入不了的问题
```java
public class xxxService {
    @Autowired
    private xxxService<A> xxAService;
    @Autowired
    private xxxService<B> xxBService;
}
```
此时xxAService和xxBService在Spring容器中的类型都是xxxService，因此注入的时候会报错，说找不到唯一可注入的bean。

因此带有发泛型参数的bean在Spring中没法按照类型注入，只能按照name注入，泛型参数经过javac后被擦除掉了，因此Spring也没法区分这二者的类型。

看到extends，我们也能想到super, ？ super T也是一个类型，这个类型要么是T要么是T的超类。

