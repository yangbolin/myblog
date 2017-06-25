title: maven自己的仲裁机制
date: 2014-06-18 22:09:25
tags: maven
categories: 开发工具
---

maven现在被广泛用来做项目管理的工具，我们经常在maven的pom文件中指定我们项目依赖的二方库，我们也会经常遇到jar包冲突，类冲突的问题。关于类冲突就是由于maven自己的仲裁机制，把应该引入的jar包给仲裁了，那么maven自己到底是如何仲裁jar包的呢？

<!-- more -->

假设我们在自己的pom.xml中引入下面的jar包
```xml
    ...
    <dependency>
        <groupId>com.xx.yy</groupId>
        <artifactId>AA</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.xx.yy</groupId>
        <artifactId>BB</artifactId>
        <version>1.0.0</version>
    </dependency>
    ....
```
AA间接依赖CC对应maven坐标如下
```xml
    <dependency>
        <groupId>com.xx.yy</groupId>
        <artifactId>CC</artifactId>
        <version>1.0.0</version>
    </dependency>
```
BB间接依赖CC对应maven坐标如下
```xml
    <dependency>
        <groupId>com.xx.yy</groupId>
        <artifactId>CC</artifactId>
        <version>1.0.1</version>
    </dependency>
```

此时maven编译的时候会进行仲裁，首先看依赖的路径，假设当前项目是X，对于CC有两条依赖路径,只是version不同，其他都一样。
1.X->AA->CC 1.0.0
2.X->BB->CC 1.0.1
发现两条路径的长度一样。接下来观察AA和BB在pom中声明的顺序，发现AA在BB的前面，此时CC使用1.0.0的版本。

因此maven自己的仲裁机制是先看路径长度，路径长度一样再看声明顺序。

如果我们书写下面的pom文件
```xml
...
    <dependency>
        <groupId>com.xx.yy</groupId>
        <artifactId>AA</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.xx.yy</groupId>
        <artifactId>AA</artifactId>
        <version>1.0.1</version>
    </dependency>
...
```

按照上面的规则，路径相同，看声明顺序，因此1.0.0的版本被使用。

注意:
maven仲裁的前置条件是artifactId和groupId一样。