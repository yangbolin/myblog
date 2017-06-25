title: JVM常用命令总结
date: 2014-09-17 19:38:19
tags: JVM
categories: JVM学习
---

#### GC相关
jstat -gcutil pid time
实时查看JAVA进程PID的垃圾回收情况，PID表示JAVA进程的ID，time表示统计的时间频率，单位是ms，比如jstat -gcutil 8888 1000 表示每隔1000ms统计一次JAVA进程8888的GC回收。

<!-- more -->

#### 内存相关
jmap -dump:format=b,file=/home/admin/xxx.bin PID 
获取JVM的内存的dump文件

jinfo -flag +HeapDumpBeforeFullGC PID
jinfo -flag +HeapDumpAfterFullGC PID
设置这两个标记后，可以让JVM在发生FGC前后自动dump堆内存，等dump完后，执行下面的命令去掉这两个标记
jinfo -flag -HeapDumpBeforeFullGC PID
jinfo -flag -HeapDumpAfterFullGC PID
在dump结束后，记得去掉这两个标记

jmap -histo:live PID
查看系统中存活的实例数目



