title: awk使用总结
date: 2015-09-09 21:12:36
tags: awk
---
####一.概述
在平时工作中经常会遇到一些文本的处理，awk就是一种文本处理语言，很方便，很轻量，可以说awk也是一种编程语言。

awk的工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，\$0则表示所有域，\$1表示第一个域，\$n表示第n个域。默认域的分隔符是"空白键"或"tab键"。
<!-- more -->

####二.常用命令
#####1.显示指定分隔符分隔后的第一列
```awk
awk -F ':' '{print $1}' /etc/passwd
```
-F ':' 指定了awk处理行文本的分隔符号
{print $1}表示是action，对于文本中的没一行都要执行这个action
这个文本处理模式是awk+action

#####2.搜索文件中包含某一关键字的行
```awk
awk -F: '/root/' /etc/passwd
```
这里使用的pattern，匹配了pattern的行才会执行后面的action，没有指定action默认输出正行内容。
```awk
awk -F: '/^root/' /etc/passwd
```
匹配所有以root开头的行

#####3.awk内置变量
```awk
awk -F ':' '{printf("filename:%s,linenumber:%s,linecontents:%s columns:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
```
FILENAME awk浏览的文件名
NR 已读出的记录数目
NF 浏览记录的域的个数
printf用于格式化输出

#####4.变量和赋值
统计一个文件中的行数
```awk
awk '{count++;print $0} END{print "user count is ", count}' /etc/passwd 
```
注意这里的count虽然没有被初始化，但是它的值是0，但是稳妥的做法还是初始化一下
```awk
awk 'BEGIN {count=0;print "[start]user count is ", count}' {count=count+1;print $0} END {print "[end] user count is ", count}' /etc/passwd
```
BEGIN后面的action只有在开始的时候才会执行，END后面的语句只有结束的时候才会执行

统计某个文件夹下的文件占用的字节数
```awk
ls -l | awk 'BEGIN {size=0} {size=size+$5} END{print "[end]size is ",size}'
```

如果以M为单位显示：
```awk
ls -l | awk 'BEGIN {size=0} {size=size+$5} END{print "[end] size is ", size/1024/1024, "M"}'
```
这个统计命令在排查线上日志超出限制时很有用。