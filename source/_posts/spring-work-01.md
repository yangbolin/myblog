title: Spring的一些小事
date: 2016-11-12 21:55:20
tags: Spring
categories: 开源框架
---

最近有同事看到下面的代码
```java
public class A implements IA {
private B b;
public void setB(B b){}
}
public class B implements IB {
}
```
Spring的配置文件如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-2.5.xsd
http://www.springframework.org/schema/tx
http://www.springframework.org/schema/tx/spring-tx-2.5.xsd"
default-autowire="byName">
<bean id="a" class="A" />
<bean id="b" class="B" />
</beans>
```
这样的代码在运行时a这个bean中的b能注入吗？因为setter方法的注入需要在配置文件中显示去写属性的
```xml
<bean id="a" class="A">
<property name="b" ref="b"/>
</bean>
```
需要这样写才可以，为啥不写也可以呢？我当时也看了半天，最后发现是XML头部schema中的
```
default-autowire="byName"
```
导致。很简单一个写法，但是一般我们不会直接在XML的头部带上这个的。

另外最近又重温了一遍Spring的源码，发现了下面这个接口中两个方法
```java
public interface ConfigurableListableBeanFactory
extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {

/**
* Ignore the given dependency type for autowiring:
* for example, String. Default is none.
* @param type the dependency type to ignore
*/
void ignoreDependencyType(Class type);

/**
* Ignore the given dependency interface for autowiring.
* <p>This will typically be used by application contexts to register
* dependencies that are resolved in other ways, like BeanFactory through
* BeanFactoryAware or ApplicationContext through ApplicationContextAware.
* <p>By default, only the BeanFactoryAware interface is ignored.
* For further types to ignore, invoke this method for each type.
* @param ifc the dependency interface to ignore
* @see org.springframework.beans.factory.BeanFactoryAware
* @see org.springframework.context.ApplicationContextAware
*/
void ignoreDependencyInterface(Class ifc);
....
}
```
看上面的注释不是很明白，ignoreDependencyType和ignoreDependencyInterface，从方法名来看一个是忽略某些类的依赖，一个是忽略某些接口的依赖。什么意思呢？在Spring中我们经常使用的是面向接口的编程，也就是在自动注入中，如果发现接口或者类被ignoreDependency了，就不会自动注入了。比如说你不能自动注入BeanFactory和ApplicationContext，它们必须通过BeanFactoryAware和ApplicationContextAware来注入，其实直接看英文注释也非常简单。




