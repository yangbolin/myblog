title: 使用xstream解析utf-8格式的字符串
date: 2015-11-18 14:26:04
tags: JAVA
categories: 编程开发
---
最近在使用xstream解析一个xml字符串时，出现了解析失败的问题，出错原因很简单，xml字符串中指定了utf-8的编码，此时我们直接构造一个
```java
XStream xstream = new XStream();
```
这样构造的话，一定会解析失败。
需要按照下面的方式来构造才能正确解析utf-8格式的xml字符串
```java
XStream xstream = new XStream(new DomDriver("UTF-8"));
```
xstream是一个比较方便的工具，能够实现xml和object之间的相互转换，注意在平时开发过程中的灵活使用。
