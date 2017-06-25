title: sqlmap被Spring容器加载了
date: 2014-07-01 20:10:20
tags: [Spring,IBatis]
categories: 编程开发
---

####一.现象
最近发布需求升级一个中间件，一个web应用在启动的时候，偶尔会会报下面的错误，报错的规律无法跟踪，导致应用启动失败。
![ibatis-psring](http://bolinyoung.qiniudn.com/ibatis-spring.png)
有时候启动应用就会出现上面的异常，有时候启动应用没有异常，在发布的时候要是机器启动失败，我们就不断重启，直到机器重启成功。

<!-- more -->

####二.分析
首先，从上面的异常看到是由于当前机器上没法访问ibatis.apache.org这个域名导致，一台机器要是没法访问某个域名，要么这台机器本身有问题，要么域名对应的机器有问题，机器本身的问题可以排除，因为ping这个域名能ping的通，那么唯一的问题就是域名对应的机器拒绝访问导致，或许ibatis.apache.org这个服务不稳定。因此，上面的UnkonwHostException:ibatis.apache.org只能解释解释域名提供的服务不稳定了。

其次，我们在所有的配置文件中找ibatis.apache.org这个域名，发现sqlmap的总控文件和每个sqlmap文件中有引用这个域名
```xml
<!DOCTYPE sqlMapConfig PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN" "http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
```
```xml
<!DOCTYPE sqlMap PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN" "http://ibatis.apache.org/dtd/sql-map-2.dtd">
```
奇怪，ibatis的配置文件怎么会被Spring解析呢？要解析也是有ibatis来解析。为什么说ibatis的配置文件被Spring解析了呢？看上图异常堆栈的开始，箭头所指


>at org.apache.xerces.parsers.DOMParser.parse(Unknown Source)
>at org.apache.xerces.jaxp.DocumentBuilderImpl.parse(Unknown Source)
>at org.springframework.beans.factory.xml.DefaultDocumentLoader.loadDocument(DefaultDocumentLoader.java:75)
>at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:396)
... 63 more

这行日志很明确地告诉我们Spring在偷偷解析的sqlmap配置文件了，注意核心关键词org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(),Spring这个时候处于bean定义的读取状态。

至此，有两个问题没有解决

1.Spring为什么会解析这份sqlmap的配置文件呢？
2.Spring解析每份XML的配置文件时，都会从网上找DTD文件吗？

先来回答问题1，Spring之所以会去偷偷解析这份配置文件，因为这份配置文件被Spring容器加载了，这份配置文件能被Spring容器加载，那是因为有人写错了，看了一下应用的总控配置文件，果然被写成这样子
```
<?xml version="1.0" encoding="GBK"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN" "http://www.springframework.org/dtd/spring-beans-2.0.dtd">
<beans>
    <import resource="classpath*:xxxx/bean/**/*.xml"/>
</beans>
```
刚好那份sqlmap的总控配置文件也写在这xxx/bean/**/*能匹配到的目录下面，这样这份配置文件必然会被Spring容器所加载了。

再看第2个问题，答案是显然不会从网上去找DTD文件，换做是你，你也不会这么写,Spring那么多DTD文件都是存在于Spring的jar包中，在XML解析的时候会把网络地址转换成classpath中某个DTD文件的路径。关键上面的域名ibatis.apache.org在Spring做XML解析的时候没法转换成自己jar包正某个DTD文件的路径，因此XML解析程序就会从网上去找这个DTD文件，这样ibatis.apache.org这个域名就会被访问了，这个过程可以看XML解析的源代码。

其实只要看看EntityResolver这个接口的注释就明白了。

####三.解决办法
1.移动sqlmap的位置，保证不会被Spring容器所加载。
2.把Spring总控配置文件中正则表达式修改一下，不要命中sqlmap即可。

####四.总结
1.最好避免sqlmap被Spring容器加载，首先这份配置文件有解析它的代码，但是不是Spring，其次Spring解析它，一个bean也解析不到，还有可能在网络不好的时候出现网络访问异常。
2.关于上面问题的排查，其实不像写的这么顺利，一开始忽略了最重要的问题：Spring为什么会去解析ibatis的配置文件呢？导致自己一直在怀疑XML jar包是不是冲突导致，另外要是有个问题很难定位的时候，注意那些曾经被我们忽略的细节点，或许问题的根源就在哪里。