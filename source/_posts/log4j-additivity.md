title: 关于log4j的additivity
date: 2014-10-31 17:10:37
tags: log4j
categories: 编程开发
---

####一.问题描述
最近优化了一下一个应用的日志，但是优化后的结果不是自己想要的，简单说指定的log部分输出到了root中的appender中，部分输出到自己指定的log中。

<!-- more -->

```xml
<appender name="BOORT" class="org.apache.log4j.DailyRollingFileAppender">
	<param name="file" value="xxx/boort.log" />
	<param name="append" value="true" />
	<param name="encoding" value="GBK" />
	<layout class="org.apache.log4j.PatternLayout">
		<param name="ConversionPattern" value="%d %-5p %c{2} - %m%n" />
	</layout>
</appender>
<appender name="OPENAPI" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="file" value="xxx/service/api.log"/>
        <param name="append" value="true"/>
        <param name="encoding" value="GBK"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d %-5p %c{2} - %m%n"/>
        </layout>
</appender>

<logger name="xxxx.Service" additivity="true">
        <level value="info"/>
        <appender-ref ref="OPENAPI"/>
</logger>
<root>
	<level value="info" />
	<appender-ref ref="BOORT" />
	<appender-ref ref="JmonitorAppender" />
</root>
```
上述xxxx.Service中的日志部分会出现到boort.log，部分会出现到api.log中。但是我们期望xxxx.Service中日志只出现到api.log中。

####二.解决方案
上面的问题肯定是日志配置的问题，google后发现additivity="true"在搞鬼
[log4j的配置](http://wiki.apache.org/logging-log4j/Log4jXmlFormat)
具体说明如下：
![log4j的additivity](http://bolinyoung.qiniudn.com/log4j-additivity.png)