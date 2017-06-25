title: Spring容器的事件机制 
date: 2014-07-07 13:08:20 
tags: Spring 
categories: 编程开发
---


####一.概述 
Spring容器暴漏了很多扩展点给框架的使用者，让框架的使用者能够干涉一些容器内部的事情，也让框架的使用能够感知到容器内部发生了什么事情，比如Spring容器在初始化完后，就会广播一个容器初始化完成的事件，然后事件的监听者监听到这个事件后做相应的事情，这也是Spring容器的一个非常有用的扩展点。

<!-- more -->

####二.事件相关的类图 

![Spring事件机制相关的类图](http://bolinyoung.qiniudn.com/Spring-Event.png)

上面这张类图就是Spring事件机制涉及到的一些核心类图。关于事件机制核心的接口有两个ApplicationEventMulticaster和ApplicationListener,前者是事件的广播中心接口，主要负责事件的广播和监听者的注册，后者是事件监听接口,还有一个事件对象ApplicationEvent，定义事件相关的属性，比如事件的类型，事件关联的数据，不过Spring中并没有定义相关的事件类型，不同的事件对应不同的类，比如ContextRefreshedEvent，表示Spring容器上下文初始化结束了。

其实事件这个概念在我们日常的开发中经常会被用到，很多人喜欢设计一个事件模块，要是让你去设计一个事件模块的话，需要考虑三个点，事件本身的定义，包括事件的类型以及事件所包含的数据，其次，需要定义事件的广播机制，一般有同步和异步，最后当然就是事件监听者的定义了，在监听者内部定义如何响应各自感兴趣的事件。

#####三.Spring事件机制的巧妙使用
Spring容器在初始化完成后，会发广播一个容器初始化OK的事件出来，我们可以在每个bean注册一个监听上下文刷新成功的事件，等上下文刷新成功了，我们在用这个bean去做一些其他事情。
```java
public class XXBean implements InitializingBean, ApplicationContextAware
    // 把Spring容器注入到这个bean中
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (applicationContext instanceof AbstractApplicationContext) {
            AbstractApplicationContext apc = (AbstractApplicationContext)applicationContext;
            // 注册新listener
            apc.addListener(new XXXApplicationListener());
        }
    }

    private final class XXXApplicationListener implements ApplicationListener {
        @Override
        public void onApplicationEvent(ApplicationEvent event) {
            if (event instanceof ContextRefreshedEvent) {
                // 这时候这个bean已经在Spring容器中初始化OK了
                XXBean.this.xxx.xxx // 使用初始化好的bean中的相关属性做一些事情
            }
        }
    }
}
```