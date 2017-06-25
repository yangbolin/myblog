title: nginx线上出错
date: 2016-10-20 13:01:10
tags: Nginx
categories:   线上运维
---
最近在做nginx的升级，很多网站的架构都是代理服务器(Apache/Nginx)加上应用容器(Tomcat/Jetty/Jboss)，Nginx在性能上由于Apache，在升级的过程中出现了下面的错误
```
nginx: [emerg] could not build the variables_hash, you should increase either variables_hash_max_size: 512 or variables_hash_bucket_size: 64
```
查找Nginx的文档，找到解决方案
```
http  {
    ......
    variables_hash_max_size 51200;
    variables_hash_bucket_size 6400;
    ......
}
```
在http的配置中增加上面两个配置即可。
另外，对于基础软件的升级一定要注意操作上的顺序，本次升级过程中发现之前老的一些配置都不兼容，如果操作不当很容易引发线上故障。
