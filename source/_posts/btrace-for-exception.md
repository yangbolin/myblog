title: 使用BTrace监控方法入参
date: 2014-10-12 18:06:18
tags: BTrace
categories: 开发工具
---
####一.现象
昨天在生产环境出现一个类的方法被调用然后出现了异常，这个异常没有被catch而是直接抛给上层的调用，我们想知道抛异常时调用该方法的参数是什么样子的，然后根据这个参数分析问题聚图出现在什么地方。

<!-- more -->

为了能够清晰的说明问题，这里大致描述一下昨天出问题的类的代码结构
```java
public class xxxUtil {
	public static Object decodeXxx(String sSrc, String sSkey,String iVParam) throws Exception {
	// 在这里做解码，然后在这里抛异常了
	}
}
```
静态方法decodeXxx被调用时，在方法内部抛出了没有被catch住的异常，这就是基本现象。

####二.排查过程
我们需要知道调用decodeXxx方法是参数是什么样子的，生成环境没法DEBUG只能考虑使用BTrace脚本了，使用BTrace动态监控decodeXxx的调用，然后输出方法调用的参数，于是我准备了如下的BTrace脚本，此时所有decodeXxx方法的调用都会被监控到，如何区分有异常的case呢？我们日志中记录的异常以及异常发生的时间，因此我们需要在监控脚本中输出监控时间，然后和日志中异常发生的时间对比，找出最终出现问题的方法调用。
```java
import static com.sun.btrace.BTraceUtils.*;
import com.sun.btrace.annotations.*;
import com.sun.btrace.AnyType;

@BTrace
public class TraceMethodArgsAndReturn{

        @OnMethod(
        clazz="xxx.xxxUtil",
        method="decodeXxx",
        location=@Location(Kind.RETURN)
        )
        public static void traceExecute(AnyType sSrc, AnyType sSkey, AnyType iVParam, @Return Object result){
                println("Call xxxUtil.decodeXxx");
                // 打印入参sSrc
                println(strcat("sSrc is:",str(sSrc)));
                // 打印入参sSkey
                println(strcat("sSkey is:",str(sSkey)));
                // 打印入参iVParam
                println(strcat("iVParam is:",str(iVParam)));
                // 打印函数返回结果result
                println(strcat("result is:",str(result)));
                // 输出时间,这里之所以输出时间就是为了和日志中的时间对比寻找出错时方法调用的入参，日志中记录了异常时间
                println(strcat("time is:",str(timeMillis()));
        }
}
```
脚本准备好了，然后在线上和JVM建立链接
```
sudo -u admin /home/boris.yangbl/btrace/bin/btrace -cp /home/boris.yangbl/btrace/build 8485 TraceMethodArgsAndReturn.java
```
其中8485是JVM的PID。
执行命令后，监控确实被监控到了，但是方法内部抛异常的case没有被监控到，奇怪，此时子写检查上面的监控脚本，发现location=@Location(Kind.RETURN)，这说明方法返回时才会被监控到，要是方法抛异常了，方法的return字节码压根就不会被调用，要是方法的return字节码不会被调用，那BTrace动态增加的字节码也不会被调用，因此需要修改一下BTrace脚本了，输出的时候不要限制方法返回，现在把BTrace脚本修改如下

```java
import static com.sun.btrace.BTraceUtils.*;
import com.sun.btrace.annotations.*;
import com.sun.btrace.AnyType;

@BTrace
public class TraceMethodArgsAndReturn{

        // 方法返回时的监控
        @OnMethod(
        clazz="xxx.xxxUtil",
        method="decodeXxx",
        location=@Location(Kind.RETURN)
        )
        public static void traceExecute(AnyType sSrc, AnyType sSkey, AnyType iVParam, @Return Object result){
                println("return xxxUtil.decodeXxx");
                // 打印入参sSrc
                println(strcat("sSrc is:",str(sSrc)));
                // 打印入参sSkey
                println(strcat("sSkey is:",str(sSkey)));
                // 打印入参iVParam
                println(strcat("iVParam is:",str(iVParam)));
                // 打印函数返回结果result
                println(strcat("result is:",str(result)));
        }
        // 进入方法的监控
        @OnMethod(
        clazz="xxx.xxxUtil",
        method="decodeXxx"
        )
        public static void traceExecute(AnyType sSrc, AnyType sSkey, AnyType iVParam, @Return Object result){
                println("enter xxxUtil.decodeXxx");
                // 打印入参sSrc
                println(strcat("sSrc is:",str(sSrc)));
                // 打印入参sSkey
                println(strcat("sSkey is:",str(sSkey)));
                // 打印入参iVParam
                println(strcat("iVParam is:",str(iVParam)));
                // 打印函数返回结果result
                println(strcat("result is:",str(result)));
        }
}
```
修改成上面的监控脚本后，我们都不需要在脚本中输出监控时间了，要是有方法抛异常的化了，必然会出现只有enter(进入方法的监控)没有return(方法返回时的监控)的输出，由于线上有两台机器，因此同时在这两台机器上使用了这个BTrace脚本，最终定位到了decodeXxx方法内部抛出异常时方法的入参是什么。后面的分析和业务有关，这里不做过多解释。

####三.最后总结
location=@Location(Kind.RETURN)要是方法内部出现异常的话，此时被BTrace修改后的字节码就没法执行了。
