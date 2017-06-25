title: Linux命令总结
date: 2014-06-13 14:30:04
tags: Linux
categories: 开发工具
---

一些常用Linux命令的总结

<!-- more -->

* lsof -p <pid> | grep xx
查看当前进程中有没有使用xx相关的东西，比如可以查看当前JVM有加载的某个jar包完整信息 ，包括版本。

* pmap <pid> | grep xx
查看当前进程中有没有使用xx相关的东西，比如可以查看当前JVM有加载的某个jar包完整信息 ，包括版本。

* df -hl
查看磁盘占用率

* du -sh 文件目录
查看当前文件的大小

* grep -R "xxx" .
在当前目录下查看包含xxx字符串的文件

* sudo killall wpa_supplicant
Linux当无线网络断掉之后，发现重新链接无线网络一直链接不上，此时执行一下上面的命令就可以了，避免每次重启电脑。

* sh -x xx.sh
调试shell脚本

* jar xvf root.war
解压war包

* jar cvf root.war
生成war包

* jar xvf web-deploy.jar
解压jar包

* jar cvf web-deploy.jar
生成jar包

* tar zxvf xxx.tgz
解压tgz的包

* find . -print | xargs fgrep "xx"
在当前文夹夹下面查找包含xx的文件

* iconv -fgbk -tutf8 xxx.log
查看有乱码的日志文件

* find `pwd` -iname "*java" -type f -exec iconv -f UTF-8 -t GBK {} -o {} \;
对.java文件进行编码转换。

* find . -iname "*.svn" -type d | xargs rm -rf
删除.svn目录

* netstat -a
查看机器的所有TCP链接，比如可以看到当前机器链接到哪个数据库上，如果机器上有应用使用了数据库的话。

* shift+$
通过VI打开文件，把光标移动到行末