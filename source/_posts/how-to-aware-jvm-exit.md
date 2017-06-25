title: JAVA代码中如何感知到JVM退出
date: 2014-06-19 22:42:58
tags: JAVA
categories: 编程语言
---

####一.需求
当JVM退出的时候，把内存中的某块数据写入到DB。分析这个需求，我们需要让应用感知到JVM准备退出了，然后把数据写入DB，然后JVM继续执行退出...

<!-- more -->

####二.实现
* 基于Spring的DisposableBean接口来实现
我们只要在自己的应用编写一个bean,实现DisposableBean这个接口中的destroy方法，然后在destroy方法中编写清理逻辑即可。要是我们不给Spring容器注册这样一个JVM关闭的钩子，JVM关闭的时候destroy方法同样不会被调用。

```java
ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("....xml");
// 注册JVM关闭时回调的钩子到Spring容器
ctx.registerShutdownHook();
```
必须在应用中给Spring容器注册这样关闭钩子，负责bean的destroy方法在JVM退出的时候是没法调用的。

另外，如果我们的应用是一个web应用，我们在停止web应用是，该应用中实现了DisposableBean这个接口的bean的destroy方法也会被调用，这个被调用的原因不是由于创建Spring容器时注册了关闭JVM回调的钩子，而是因为关闭web应用时会回调ServletContextListener的contextDestroyed方法，ServletContextListener的实现类是ContextLoaderListener，contextDestroyed的实现如下

```java
public class ContextLoaderListener implements ServletContextListener {
    private ContextLoader contextLoader;
    ...
	public void contextDestroyed(ServletContextEvent event) {
		if (this.contextLoader != null) {
		    //  关闭Spring容器
			this.contextLoader.closeWebApplicationContext(event.getServletContext());
		}
	}
    ...
}
```
我们再看看ContextLoader的closeWebApplicationContext方法实现

```java
public class ContextLoader {
    //....
	public void closeWebApplicationContext(ServletContext servletContext){
		servletContext.log("Closing Spring root WebApplicationContext");
		try {
			if (this.context instanceof ConfigurableWebApplicationContext){
			    // 销毁servlet的上下文，调用Spring容器的close方法
			    ((ConfigurableWebApplicationContext) this.context).close();
			}
		}
		finally {
		    // ...
		}
	}
	//...
}
```

因此web应用中不需要给Spring容器注册JVM关闭的钩子，代码中依然能够感知到JVM的关闭。

* 直接基于Runtime的addShutdownHook方法来实现

```java
Runtime.getRuntime().addShutdownHook(new Thread(){

    @Override
    public void run() {
        System.out.println("JVM EXIT...");
    }
});
```
JVM退出的时候会回调我们注册的这个钩子，Spring注册JVM关闭的回调也是这么搞的，具体AbstractApplicationContext.java中源代码如下。

```java
public class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext, DisposableBean {
	....
		/**
	 * Register a shutdown hook with the JVM runtime, closing this context
	 * on JVM shutdown unless it has already been closed at that time.
	 * <p>Delegates to <code>doClose()</code> for the actual closing procedure.
	 * @see java.lang.Runtime#addShutdownHook
	 * @see #close()
	 * @see #doClose()
	 */
	public void registerShutdownHook() {
		if (this.shutdownHook == null) {
			// No shutdown hook registered yet.
			this.shutdownHook = new Thread() {
				public void run() {
					doClose();
				}
			};
			Runtime.getRuntime().addShutdownHook(this.shutdownHook);
		}
	}
	....
}
```

####三.总结
* 回调思想
回调也是实现程序解耦的一种思路，在平时开发的过程中要注意回调的使用，回调的调用面向接口，回调的实现交给第三方来实现，我们也可提供缺省的回调实现。

* 可扩展性
自己实现的框架或一定要提供可扩展的地方，方便满足别人个性化的一些需求。