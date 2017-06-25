title: 使用svn merge命令来回滚代码
date: 2015-07-02 09:18:16
tags: svn
categories: 开发工具
---
####一.问题
我们经常使用svn来托管代码，svn ci就能把代码提交到svn服务器上，当我们误操作使用了svn ci后，此时想回滚怎么办？如果把本地代码回到执行svn ci之前的状态？

<!-- more -->

####二.解决方案
* 1.svn up
查看提交后的版本，假设为M
* 2.svn log --stop-on-copy 
查看svn提交的日志，选择要回滚到那个版本，假设为N
* 3.svn diff -r M:N
查看从版本M到版本N发生了那些变化
* 4.svn merge -rM:N svn-branch
从版本M回滚到版本N svn-branch就是当前svn的分支URL
* 5.修改好本地文件后再次提交
