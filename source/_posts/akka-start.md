title: Akka入门
date: 2014-10-25 22:35:44
tags: Akka
categories: 并发技术
---
#### 一.概述
为了提高一个数据计算平台的吞吐了，最近抽空看了看akka，之前听同事说akka能够提高并发吞吐量，高容错，很稳定，今天抽时间先简单看了相关的文档，写了个简单的例子，主要演示如何使用akka，当然我们也可以搭建一个akka的集群。

<!-- more -->

#### 二.快速开始

akka相关的依赖
```xml
<!--  akka starting -->
<dependency>
	<groupId>com.typesafe.akka</groupId>
	<artifactId>akka-actor_2.10</artifactId>
	<version>2.2.3</version>
</dependency>
<dependency>
	<groupId>com.typesafe.akka</groupId>
	<artifactId>akka-remote_2.10</artifactId>
	<version>2.2.3</version>
</dependency>
<dependency>
	<groupId>com.typesafe.akka</groupId>
	<artifactId>akka-kernel_2.10</artifactId>
	<version>2.2.3</version>
</dependency>
<dependency>
	<groupId>com.typesafe.akka</groupId>
	<artifactId>akka-cluster_2.10</artifactId>
	<version>2.2.3</version>
</dependency>
<dependency>
	<groupId>com.typesafe.akka</groupId>
	<artifactId>akka-contrib_2.10</artifactId>
	<version>2.2.3</version>
</dependency>
<dependency>
	<groupId>com.typesafe.akka</groupId>
	<artifactId>akka-slf4j_2.10</artifactId>
	<version>2.2.3</version>
</dependency>
<dependency>
	<groupId>org.fusesource</groupId>
	<artifactId>sigar</artifactId>
	<version>1.6.4</version>
</dependency>
<dependency>
	<groupId>com.typesafe.akka</groupId>
	<artifactId>akka-cluster_2.10</artifactId>
	<version>2.2.3</version>
</dependency>
<!--  akka ending  -->
```
java代码
```java
public enum Msg {
    GREET, DONE, WELCOME;
}

public class Greeter extends UntypedActor {
    @Override
    public void onReceive(Object msg) throws Exception {
        if (msg == Msg.GREET) {
            System.out.println("Greeter Messgae!");
            // 收到GREET消息后发出一个DONE的应答
            getSender().tell(Msg.DONE, getSelf());
        }
    }
}

public class Welcome extends UntypedActor {

    @Override
    public void onReceive(Object arg0) throws Exception {
        if (arg0 == Msg.WELCOME) {  
            System.out.println("Welcome Messgae!");
            // 收到WELCOME消息后，发出一个WELCOME的应答
            getSender().tell(Msg.WELCOME, getSelf());
        }
    }
}

public class HelloWorld extends UntypedActor {

    @Override
    public void preStart() {
        // 发送WELCOME消息给welcome这个Actor
        final ActorRef welcome = getContext().actorOf(Props.create(Welcome.class), "welcome");
        welcome.tell(Msg.WELCOME, getSelf());
        
        // 发送GREET消息给greet这个Actor
        final ActorRef greeter = getContext().actorOf(Props.create(Greeter.class), "greeter");
        greeter.tell(Msg.GREET, getSelf());
    }

    @Override
    public void onReceive(Object msg) throws Exception {
        if (msg == Msg.DONE) {
            System.out.println("Message DONE");
            getContext().stop(getSelf());
        } else {
            System.out.println(msg);
        }
    }
}
```
这里的HelloWorld，Welcome，Greeter是三个Actor，在上述代码中HelloWorld向Welcome，Greeter发消息，Welcome，Greeter收到消息后分别向HelloWorld发出回应。
HelloWorld在收到DONE消息后会终止自己。通过akka.Main来启动，设置启动参数com.bolin.young.akka.HelloWorld，这样就能看到相关的输出了。

####三.总结
通过上面这个小例子，我们发现akka中有一个核心的东西，那就是actor，actor之间通过消息通信，actorA发消息给actorB，actorB收到actorA的消息后，也可以对actorA发送一个回应的消息。
