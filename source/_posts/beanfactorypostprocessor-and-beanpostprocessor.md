title: BeanFactoryPostProcessor和BeanPostProcessor
date: 2014-06-24 23:29:37
tags: Spring
categories: 编程开发
---


一.概述
我们经常使用Spring框架，Spring帮我托管bean的创建以及bean的管理，同时又暴漏出一些可扩展的地方，方面程序员去干涉bean或者Spring容器的创建。今天有同学问BeanPostProcessor相关的东西，由此很容易想到BeanFactoryPostProcessor，这里总结一下，方便后面在开发的过程中灵活使用。

<!-- more -->

二.BeanPostProcessor前置处理器
Spring容器在创建bean的时候，会看有没有BeanPostProcessor，如果有的话，会回调BeanPostProcessor接口中的两个方法，在bean创建之前执行程序员的一些个性化代码，在bean创建之后也执行一些程序员的个性化代码。
```
public interface BeanPostProcessor {
    /** 创建bean之前回调 **/
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
	/** 创建bean之后回调 **/
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

三.BeanFactoryPostProcessor
这个可以让程序员去干涉一下Spring容器的创建，Spring容器创建的时候会回调这个接口的定义的这个方法，比如你可以在实现了这个接口的方法中给Spring容器注册一个JVM关闭时回调的钩子。
```
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor{

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        // 注册JVM关闭时的回调
        ConfigurableApplicationContext cxt = (ConfigurableApplicationContext)beanFactory;
        cxt.registerShutdownHook();
    }
}
```