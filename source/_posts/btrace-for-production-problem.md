title: 使用BTrace来定位方法中Catch住的异常
date: 2014-05-14 15:55:55
tags: [JAVA,BTrace]
categories: 开发工具
---

####一.问题
今天在自己应用中引入了一个新的中间件，发布到线上后，发现一个接口一直获取不到数据，此时找不到日志，因为代码中的日志输出不按常理出牌，误引入了某一中间件自定义的Logger导致日志很难找到，此时要是在测试环境，debug一下就搞定，但是线上不能这么搞，如何去查看一下本次调用接口返回的数据到底是什么呢？

<!-- more -->

####二.被调用到的代码
![被调用到的代码](http://bolinyoung.qiniudn.com/udc.png)

上面代码的getData方法被调用了，但是返回的DataResult中没有数据。

####三.使用BTrace监控一下getData方法的返回值以及输入参数
```java
import static com.sun.btrace.BTraceUtils.*;
import com.sun.btrace.annotations.*;
import com.sun.btrace.AnyType;

@BTrace
public class TraceMethodArgsAndReturn{

        @OnMethod(
        clazz="xxx.UdcDataSource",
        method="getData",
        location=@Location(Kind.RETURN)
        )
        public static void traceExecute(@Self Object dataSource, AnyType context, @Return Object result){
                println("Call UdcDataSource.getData");
                // 打印当前被BTrace拦截到的实例
                println(strcat("dataSource is:",str(dataSource)));
                // 打印getData方法的参数 xx.DefaultParamContext是context的类型
                println(strcat("fields are:",str(get(field("xx.DefaultParamContext","fields"),context))));
                println(strcat("context are:",str(get(field("xx.DefaultParamContext","context"),context))));
                // 打印getData的返回值 xx.DataResult是result的实际类型
                println(strcat("result is:",str(get(field("xx.DataResult","result"),result))));
        }
}
```

使用上面这段BTrace脚本，就可以跟踪getData方法的调用了，跟踪的结果是getData方法返回DataResult实例，但是DataResult实例中的Map实例大小为0,也就是说上面代码中ret大小为0。

仔细看看上面调用到的代码，如果返回结果是这样的话，只能说明getData方法中抛异常了。看看catch语句中的日志，竟然没有把exception也输出到log里面，那如何看这里面的异常呢？难道要再发布一次。。。。要是BTrace也能把这个异常跟踪到，那就好办了。

在BTrace脚本中不能出现自己定义的类型，要是自己定义的类类型，要么使用AnyType,要么使用Object在BTrace脚本中来指定。

####三.使用BTrace来跟踪方法中catch住的异常
不清楚怎么搞，只能google一下，看了一些BTrace的文档，发现BTrace竟然能跟踪到方法中catch住的异常。于是写了下面的代码

```java
import static com.sun.btrace.BTraceUtils.*;
import com.sun.btrace.annotations.*;

@BTrace
public class TraceException {
   // 异常捕捉
   @OnMethod(clazz = "xxx.UdcDataSource", method = "getData", location = @Location(Kind.CATCH))
   public static void traceExecute(@ProbeClassName String pcn, @ProbeMethodName String pmn, Exception e) {
      print(pcn);
      print(".");
      print(pmn);
      print("(");
      println(") cacth Exception");
      println(e);
   }
}
```
使用Kind.CATCH就能捕获方法中catch住的异常了，至此异常发现，问题搞定。

####五.BTrace使用
关于BTrace的使用很简单，可以参考http://jm-blog.aliapp.com/?p=509

####六.需要注意的问题
我们知道BTrace是动态修改类的字节码的，动态增加一些字节码，那么这些动态增加的字节码在BTrace断开后，会被删除吗？也就是说被修改了的类能恢复到修改之前吗？答案是不能，在BTrace断开后，BTraceRuntime就会记录自己已经禁用了,把disabled设置为true，于是虽然被改写的字节码还保留着对它的调用，但BTraceRuntime看到disabled为true就直接返回了.

```java
/**
 * Enter method is called by every probed method just
 * before the probe actions start.
 */

public static boolean enter(BTraceRuntime current) {
	if (current.disabled) return false;
	return map.enter(current);
//        // check we have entered already or disabled
//        if (current.disabled || (tls.get() != null)) {
//            return false;
//        } else {
//            tls.set(current);
//            return true;
//        }
}

// 这个方法在BTrace退出的时候会被调用
private synchronized void exitImpl(int exitCode) {
	if (exitHandler != null) {

	try {
		exitHandler.invoke(null, exitCode);
	    } catch (Throwable ignored) {
 	    }
         }

	// 退出的时候把disabled设置成true
	disabled = true;
	// ...后面的代码省略

}
```
[BTraceRuntime源码](https://kenai.com/projects/btrace/sources/hg/content/src/share/classes/com/sun/btrace/BTraceRuntime.java?rev=462)
