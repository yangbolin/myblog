title: maven dependencies 和 dependencyManagement的区别
date: 2014-05-17 17:35:04
tags: maven
categories: 开发工具
---

曾经有人问我maven中dependencies和dependencyManagement的区别，我当时主要从版本控制以及依赖引入这两方面来回答

<!-- more -->

* 版本控制
我们一般在顶级pom中使用dependencyManagement来管理一组二方库的依赖，如下所示

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.xxx.xxx</groupId>
            <artifactId>yyyxxx</artifactId>
            <version>1.0.6-SNAPSHOT</version>
        </dependency>
        ....
    </dependencies>
</dependencyManagement>
```
我们看到dependencyManagement需要dependencies来管理一组二方库的依赖，写在这里的二方库必须指定groupId,artifactId以及version，但是不一定会被应用真正依赖进来，我们最终会在子pom中写

```xml
<dependencies>
    <dependency>
        <groupId>com.xxx.xxx</groupId>
        <artifactId>yyyxxx</artifactId>
    </dependency>
    ....
</dependencies>
```
来真正引入一个二方库到我们的应用中，同时这个引入二方库的版本就是我们在顶级pom中指定的版本，当然在子pom中也可以指定自己的版本，这样就不会使用dependencyManagement中指定的version了。

* 依赖引入
写在dependencyManagement中的依赖不一定会引入到应用中，只有在子pom中通过dependencies显式指定后才会真正引入到应用中。


> 注：
> 目前暂时想到这两个点，后续发现其他点后再补充
