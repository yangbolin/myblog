title: 如何获取当前JAVA进程的PID
date: 2014-06-14 09:24:39
tags: JAVA
categories: 编程开发
---

####一.需求
如何在JAVA代码中获取当前运行JAVA进程的进程ID，通常我们可以在JVM外部执行jps命令看到某一JAVA进程的PID，但是如何在JVM内部获取这个PID呢？

<!-- more -->

####二.实现
```java
/**
 * 获取java进程PID
 * 
 * @return
 */
public static int getPID() {
    String rtName = ManagementFactory.getRuntimeMXBean().getName();
    int index = rtName.indexOf("@");
    if (index != -1) {
        return Integer.parseInt(rtName.substring(0, index));
    }

    return -99;
}
```