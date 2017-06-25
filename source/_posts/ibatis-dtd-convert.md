title: IBatis DTD 文件的转换
date: 2014-05-17 20:26:28
tags: IBatis
categories: 编程开发
---

####一.概述
我们经常使用IBatis这个ORM框架来操作数据库，在使用IBatis的时候我们需要编写sqlMap以及sqlMapConfig的配置文件。

<!-- more -->

在编写sqlMap配置文件的时候，我们需要写下面的DTD

```xml
<!DOCTYPE sqlMap PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN" "http://ibatis.apache.org/dtd/sql-map-2.dtd">
```

在编写sqlMapConfig配置文件的时候，我们需要写下面的DTD
```xml
<!DOCTYPE sqlMapConfig PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN" "http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
```

在解析XML文件的时候，会按照上面指定的DTD对XML文件进行语法分析，看看XML文件是否有语法错误。目前解析XML文件主要有两种方式DOM和SAX,前者基于DOM树，后者基于事件以及事件的监听来解析XML文件。

注意：
在写DTD时，为了避免一些诡异问题出现，要么都使用ibatis.apache.org要么都使用ibatis.com，不要二者混合起来使用。

####二.问题
IBatis框架在解析XML配置的时候会在网上去找DTD文件吗？因为配置文件中写的是一个网络地址。

如果你是实现者，你肯定不会在网上去找DTD,你必须要想办法从本地去找DTD，因为很多生产环境会有网络限制的，同时从网上去找，加载速度取决于网速。那么IBatis是怎么从本地去找的呢？

####三.IBatis把网络地址转换成本地文件路径
IBatis在解析XML配置的时候，把配置文件中的网络地址会转换成一个classpath中的文件路径，然后从classpath中加载相应的DTD文件。如下图所示
![IBatis DTD 文件转换](http://bolinyoung.qiniudn.com/IBatisDTD.png)

找找IBatis的源代码，我们就会发现IBatis二方库就中就存在sql-map-2.dtd和sql-map-config-2.dtd这两个DTD文件，同时也会把配置文件中的网络地址转换成本地文件路径。因此IBatis框架本身并不会从网络上去加载相应的DTD文件。