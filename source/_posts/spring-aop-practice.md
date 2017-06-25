title: 如何不用修改原来的代码呢？
date: 2014-04-15 19:58:15
tags: Spring
categories: 编程开发
---

####一.需求描述
最近遇到这样一个需求，有个数据计算的方法，我们需要优化一下，因为有些数据在一段时间内计算出的结果总是一样的，为了提高性能，我们把这些数据的计算结果缓存起来，这样下次计算的时候就先去缓存中查一下，要是缓存中存在，就直接把缓存中的数据返回，要是缓存中不存在就走原来的计算逻辑去计算，面对这个需求，我们需要做两件事情，在调用计算方法之前读取一下缓存，在调用方法之后把值写入缓存。这样的计算方法有10多个，相应的类也有10多个。

<!-- more -->

####二.思路分析
面对这样的需求，绝对不能去修改10多个类，首先这样做有很多工作量，其次你所做的事情都是Repeat Yourself,没意义的。其实这种需求，我们最容易想到的就是AOP理念了，定义两个切面，方法执行前执行一下，方法返回前执行一下就好，为了好描述，我们暂且认为读缓存的切面是A,写缓存的切面是B,仔细分析一下，我们执行完切面A后，可能不需要再继续执行原来的计算方法了，直接返回就完事了，要是我们把读缓存的单独抽象出一个切面来，我们在这个切面中根本没法控制不去执行被拦截的计算方法，因此我们不能把读取缓存这个逻辑单独抽象成一个Spring AOP的切面，所以上面抽象出A,B切面的思路不可行，这时候我们需要正对这些方法定义一个统一的MethodInterceptor去拦截一下这个方法，在执行方法之前去读一下缓存，执行方法之后写一下缓存，这样写出来的伪代码如下：
```java
public class XXXMethodInterceptor implements MethodInterceptor {

    private static final String METHOD_NAME = "xxxx";

    @Override
    public Object invoke(MethodInvocation arg) throws Throwable {
        // 对xxxx方法做拦截
        if (METHOD_NAME.equals(arg.getMethod().getName())) {
                // 读取缓存，要是缓存中有值，直接返回
                Object cacheResult = readFromCache();
                if (cacheResult != null) {
                    return cacheResult;
                }
                // 调用原来的方法
                Object result = arg.proceed();
                // 写缓存,当然有些计算结果是不用写缓存的
                writeCache(result);
                // 返回结果
                return resultFromDataSource;
            }
        }
        return arg.proceed();
    }
}
```

这样我们的方法拦截器就定义好了，如何把它织入到原来的代码逻辑中，我们使用Spring的ProxyFactoryBean,这是一个代理的工厂bean,它可以返回被代理的bean，同时把我们的拦截逻辑织入到代理bean中，生成代理对象的时候如果我们的类实现了接口，就会使用jdk的动态代理，重新生成一个代理类，要是我们的类没有实现接口，就会使用cglib动态去修改字节码。下面我们来看看如何织入我们的拦截逻辑
```xml
<!-- 定义抽象的代理bean -->
<bean id="abstractProxy" class="org.springframework.aop.framework.ProxyFactoryBean" abstract="true">
	<property name="interceptorNames">
		<list>
			<value>xXXXMethodInterceptor</value>
		</list>
	</property>
</bean>
<!-- 拦截器的定义 -->
<bean id="xXXXMethodInterceptor" class="xxxx.xxx.XXXMethodInterceptor" />
<!-- 新定义一个bean 织入我们的拦截器 -->
<bean id="xxxx" parent="abstractProxy">
	<property name="target">
		<bean class="xxxxxxx.xxxxx" />
    </property>
</bean>
```
注意:

* 1.我们可以在ProxyFactoryBean中定义多个拦截器，目前只是定义了一个方法的拦截器
* 2.在写Spring配置文件的时候注意抽象bean的灵活使用
