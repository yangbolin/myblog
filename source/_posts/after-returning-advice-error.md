title: 一个在方法返回前使用AOP的BUG分析
date: 2014-05-03 14:40:00
tags: [JAVA,Spring]
categories: 编程开发
---

####源代码
最近使用AOP拦截一个方法的返回值，并且修改一下方法的返回值，然后再返回，于是写了下面的代码：

<!-- more -->

![源代码](http://bolinyoung.qiniudn.com/aop-error.png)

从上图可以看出这是一个AOP的切面，在getData方法返回前对returnValue做相关的修改，然后再返回。

我们在写JAVA代码的时候经常会考虑NPE，我们发现这里也对returnValue考虑了NPE，看看红色框内代码，如果returnValue==null，就重新创建一个returnValue，其实问题就是出现在这里，我们在切面中把returnValue的指向修改了，这个修改只在afterReturning方法内部有效，出了这个方法后，这个修改就无效了，因为在afterReturning方法内部保存了一份returnValue的引用值拷贝，修改这份引用值的拷贝，方法外不会感知到，通过这份应用值拷贝去修改其指向内存空间的值，方法外能感知到内存值的修改。因此上述红色框内的代码是无效的，要是returnValue本身为null，执行完afterReturning方法后依旧是null。

关于函数参数传递的分析，参考之前的一片文章[Java 函数参数引用思考](http://yangbolin.github.io/myblog/2014/04/08/java-parameter-reference/)
