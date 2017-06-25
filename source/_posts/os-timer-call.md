title: 操作系统定时调度
date: 2015-05-18 20:13:29
tags: crontab
categories: 编程开发
---
####一.概述
最近需要开发一些定时任务，提到定时任务，我们都会想到quartz，为了让我们的定时任务能够更加灵活的被控制，我们通过shell脚本去执行一个java类，然后定时去执行这个shell脚本即可。如何去定时执行这个shell脚本，我们想到Linux本身提供后台进程去定时执行一些命令，那就是crontab，因此我们编写好自己的shell脚本再写好crontab后就能定时执行我们的任务了。

<!-- more -->

####二.如何配置
* 查看crontab的配置
crontab -l
* 编辑crontab的配置
crontab -e
* crontab的格式说明
![crontab格式说明](http://bolinyoung.qiniudn.com/crontab.jpg)

* less  /var/log/cron
查看crontab运行时的日志

####三.注意点
```shell
* 6 * * * sh xx.sh
```
表示每天凌晨6点开始没分钟执行一次sh xx.sh脚本

```shell
0 6 * * * sh xx.sh
```
表示每天凌晨6点开始执行一次sh xx.sh脚本，只执行一次

注意上面这两个频率的区别，排查问题的时候注意考虑那些一直被忽略的点，有可能那些点就是解决问题的关键之所在。