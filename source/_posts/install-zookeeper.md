title: 一台机器上面如何部署3个zookeeper的server
date: 2014-04-23 11:40:49
tags: zookeeper
categories: 分布式
---

今天在自己的电脑上玩了一下zookeeper,这里主要记录一下安装的过程，自己对zookeeper的了解还不够，没法写一些比较深入的东西。

<!-- more -->

* 1.在本地建立三个文件夹，比如server1,server2,server3如下图所示
  ![建立文件夹](http://bolinyoung.qiniudn.com/zk3files.png)

* 2.在每个文件夹下面建立data,dataLog以及logs这三个文件,进入data文件，建立myid文本文件，如果data在server1下面，则在myid中写1，如果data在server2下面，则在myid中写2，如果data在server3下面，则在myid中写3。

* 3.分别把zookeeper的安装配置包下载到server1,server2,以及server3三个文件夹下面，下载链接，[zookeeper](http://mirrors.hust.edu.cn/apache/zookeeper/stable/zookeeper-3.4.6.tar.gz)。

* 4.解压zookeeper到server1,server2,server3三个文件夹下面，然后分别在zookeeper中conf文件夹下面增加zoo.cfg，如果是server1下面的zookeeper则zoo.cfg中的内容为

```
tickTime        =2000
initLimit       =5
syncLimit       =2
##指定server1下面的data和dataLog文件
dataDir         =/home/yangbolin/zookeeper/server1/data
dataLogDir      =/home/yangbolin/zookeeper/server1/dataLog

##指定自己的端口号
clientPort      =2181

###指定zookeeper集群中的其他机器
server.1        =127.0.0.1:2888:3888
server.2        =127.0.0.1:2889:3889
server.3        =127.0.0.1:2890:3890
```
如果是server2下面的zookeeper，则zoo.cfg中的内容为
```
tickTime        =2000
initLimit       =5
syncLimit       =2
##指定server2下面的data和dataLog文件
dataDir         =/home/yangbolin/zookeeper/server2/data
dataLogDir      =/home/yangbolin/zookeeper/server2/dataLog

##指定自己的端口号
clientPort      =2182

###指定zookeeper集群中的其他机器
server.1        =127.0.0.1:2888:3888
server.2        =127.0.0.1:2889:3889
server.3        =127.0.0.1:2890:3890
```
如果是server3下面的zookeeper，则zoo.cfg中的内容为
```
tickTime        =2000
initLimit       =5
syncLimit       =2
##指定server3下面的data和dataLog文件
dataDir         =/home/yangbolin/zookeeper/server3/data
dataLogDir      =/home/yangbolin/zookeeper/server3/dataLog

##指定自己的端口号
clientPort      =2183

###指定zookeeper集群中的其他机器
server.1        =127.0.0.1:2888:3888
server.2        =127.0.0.1:2889:3889
server.3        =127.0.0.1:2890:3890
```
* 5.进入server1，然后进入到zookeeper的目录下面，再进入到bin目录下面，启动zookeeper,执行下面的脚本

```
./zkServer.sh start

```
* 6.进入server2,server3把5中的事情都做一遍，这样我们自己搭建的zookeeper集群就启动好了。

* 7.从server1,server2,server3中随便找一个文件夹进入，然后进入到zookeeper所在的文件夹，进入bin目录，执行下面脚本

```
./zkCli.sh
```
此时就可以进入到zookeeper的命令行模式，然后你就可以使用zookeeper的命令读写zk节点上的数据了。关于zookeeper的命令后面再详细介绍。

这样我们就在本地建立一个zookeeper的集群，为后续的深入研究奠定了基础。
