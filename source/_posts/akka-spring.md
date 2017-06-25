title: akka如何和Spring集成起来
date: 2014-11-17 22:08:04
tags: Akka
categories:
---

####一.概述
最近又重新学习了一下akka，并且在一个web应用中成功使用了akka提升了系统的性能。我们在web应用中一般都会有dao&&service，这些都是spring的bean，我们定义一个actor，这个actor中可能调用dao读写数据库，也可能调用service做业务逻辑，因此我们就希望actor也是从spring容器中获取到的，这样就可以在actor中注入service&&dao了。

<!--more-->

####二.如何从Spring容器中获取actor
1.用于从Spring容器中获取actor
```java
public class SpringActorProducer implements IndirectActorProducer {

    final ApplicationContext applicationContext;
    final String             actorBeanName;

    public SpringActorProducer(ApplicationContext applicationContext, String actorBeanName){
        this.applicationContext = applicationContext;
        this.actorBeanName = actorBeanName;
    }

    @Override
    public Actor produce() {
        return (Actor) applicationContext.getBean(actorBeanName);
    }

    @SuppressWarnings("unchecked")
    @Override
    public Class<? extends Actor> actorClass() {
        return (Class<? extends Actor>) applicationContext.getType(actorBeanName);
    }
}
```

2.扩展actor的创建
```java
public class SpringExtension extends AbstractExtensionId<SpringExtension.SpringExt> {

    /**
     * The identifier used to access the SpringExtension.
     */
    public static SpringExtension SpringExtProvider = new SpringExtension();

    /**
     * Is used by Akka to instantiate the Extension identified by this ExtensionId, internal use only.
     */
    @Override
    public SpringExt createExtension(ExtendedActorSystem system) {
        return new SpringExt();
    }

    /**
     * The Extension implementation.
     */
    public static class SpringExt implements Extension {

        private volatile ApplicationContext applicationContext;

        /**
         * Used to initialize the Spring application context for the extension.
         * 
         * @param applicationContext
         */
        public void initialize(ApplicationContext applicationContext) {
            this.applicationContext = applicationContext;
        }

        /**
         * Create a Props for the specified actorBeanName using the SpringActorProducer class.
         * 
         * @param actorBeanName The name of the actor bean to create Props for
         * @return a Props that will create the named actor bean using Spring
         */
        public Props props(String actorBeanName) {
            return Props.create(SpringActorProducer.class, applicationContext, actorBeanName);
        }
    }
}
```

3.示例actor以及actor依赖的service
```java
public class CountingService {
  /**
   * Increment the given number by one.
   */
  public int increment(int count) {
    return count + 1;
  }
}
class CountingActor extends UntypedActor {

    public static class Count {}
    public static class Get {}

    // the service that will be automatically injected
    private CountingService countingService;

    private int count = 0;

    @Override
    public void onReceive(Object message) throws Exception {
        System.out.println(countingService);
        if (message instanceof Count) {
            count = countingService.increment(count);
        } else if (message instanceof Get) {
            getSender().tell(count, getSelf());
        } else {
            unhandled(message);
        }
    }

    public void setCountingService(CountingService countingService) {
        this.countingService = countingService;
    }
    
    public CountingActor() {
        System.out.println("CountingActor is Creating...");
    }
}
```
CountingActor依赖了CountingService，CountingService到时候会被注入到CountingActor中。

4.配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-2.0.xsd"
	default-autowire="byName">

	<bean id="countingService" class="com.bolin.young.akka.spring.CountingService" />
	<bean id="countingActor" class="com.bolin.young.akka.spring.CountingActor"  scope="prototype" />

	<bean id="myActorSystem" class="akka.actor.ActorSystem"
		factory-method="create" destroy-method="shutdown" scope="singleton">
		<constructor-arg value="mySpringAkkaSystem" />
	</bean>
	
</beans>
```

5.如何使用
```java
// create a spring context and scan the classes
    ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

    ActorSystem myActorSystem = (ActorSystem)ctx.getBean("myActorSystem");
    
    System.out.println(myActorSystem);
    
    SpringExtProvider.get(myActorSystem).initialize(ctx);
    
    final ActorRef myActor = myActorSystem.actorOf(
                   SpringExtProvider.get(myActorSystem).props("countingActor"), "countingActor");
```

####三.最后总结
1.一般actorA发消息给actorB，actorB返回消息给actorA。我们可以自己在代码中发消息给actorA，然后坐等actorA的返回，如下面的代码
```java
BootstrapAnalyzeMsg bootstrapAnalyzeMsg = new BootstrapAnalyzeMsg();
bootstrapAnalyzeMsg.setFileItem(treeFile);

Timeout timeout = new Timeout(Duration.create(TIME_OUT, "seconds"));
Future<Object> future = Patterns.ask(bootstrapAnalyzeActor, bootstrapAnalyzeMsg, timeout);

try {
     // 当前线程会阻塞，直到有当前的Actor有消息过来。
     ProjectAnalyzeResultMsg projectAnalyzeResultMsg = (ProjectAnalyzeResultMsg) Await.result(future,                                            timeout.duration());
     return projectAnalyzeResultMsg.getJarConflictInfoList();
} catch (Exception e) {
     e.printStackTrace();
} finally {
     bootstrapAnalyzeActor.tell(new ActorStopMsg(), null);
}
```
该思想类似于java的Future。
2.actor也可以终止自己,没用的actor尽量提前终止。
```java
getContext().stop(getSelf());
```
3.actor在系统中的名字必须唯一。

4.自己动手写了一个完整的小例子 https://github.com/yangbolin/akka-demo