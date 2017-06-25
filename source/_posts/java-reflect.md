title: JAVA反射调用总结
date: 2015-06-08 22:48:54
tags: JAVA
categories: 编程开发
---

####一.概述
这几天一直在忙于一个框架的开发，我们的初衷是基于配置产出业务数据，不要让开发工程师去编写JAVA代码来产出账单，一开始觉得这件事情很难，很难做到不开发JAVA代码，事实确实如此，不过数据如果规整的话，基于配置完全可以，一行java代码都不用写。当然这个框架目前还在测试中，核心功能已经开发结束了，开发一个框架和开发一个业务功能要考虑的事情完全不一样，在开发框架之前，需要把所有可能出现的需求都要考虑一下，其实框架就是高度的抽象，我们把平时所做的一些功能逻辑梳理清楚，再上一个高度就能梳理出一个框架。在开发这个框架的过程中用到了反射，感觉JDK的反射写起来代码有点多，于是考虑用Spring框架提供反射工具类，还有木有其他处理反射调用框架或者工具呢？

<!-- more -->

####二.概述
#####1.利用JDK本身的API来实现反射
```java
Object oo = new InnerObject();
Method[] methods = oo.getClass().getDeclaredMethods();
for (Method method : methods) {
    if (method.getName().equals("test")) {
        try {
            Object retValue = method.invoke(oo, null);
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```
一个反射调用要写这么多代码，我也是醉了。。。。
#####2.利用Spring的工具类来实现反射
```java
Method method = ReflectionUtils.findMethod(oo.getClass(), "test", null);
try {
    Object retValue = method.invoke(oo, null);
} catch (IllegalArgumentException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```
虽然木有了for循环，但是代码还是有些多，不够精简。
#####3.利用Mirror来做反射
```xml
<dependency>
 	<groupId>net.vidageek</groupId>
 	<artifactId>mirror</artifactId>
 	<version>1.6.1</version>
</dependency>
```
```java
Object oo = new InnerObject();
Object retValue = new Mirror().on(oo).invoke().method("test1").withoutArgs();
```
看看mirror是不是精简了很多呢？还是函数编程思想，这种写法在JDK8中会非常普遍。这里只是举例说明了一下方法调用，其他更多反射调用参考 [mirror](http://projetos.vidageek.net/mirror/mirror/)

####三.总结
对于一个点，多思考，寻求最简单的写法，你会有更多的收获。
