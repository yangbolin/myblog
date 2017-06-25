title: maven的一些注意点
date: 2014-05-17 19:39:59
tags: maven
categories: 开发工具
---

####一.概述
目前maven被广泛用于java项目的开发中，统一管理项目依赖，以及项目的打包部署等，我们经常需要编写maven的pom文件，包括顶级pom以及子pom。maven既可以用来构建单工程项目，也可以用来构建多工程项目，子工程和子工程之间有相互依赖。

<!-- more -->

同时maven本身也提供了插件机制，使得工程师可以编写自己的插件，然后在maven的某一阶段执行自己的插件做相关的事情,后面我会开发一个maven的插件，来检查java代码的规范。

####二.一些注意点
* 顶级pom中注意modules中module的顺序
在顶级pom中，我们经常写一个modules标签，管理我们当前工程下面有多少个模块例如
```xml
<modules>
    <module>A</module>
    <module>B</module>
    <module>C</module>
</modules>
```
上面的modules说明当前工程有A,B,C三个子工程，并且install的顺序是A,B,C，这样要是A依赖B的话，执行mvn clean install必然报错，因为install A的时候，B还没有生成，所以我们在写modules的时候一定要注意顺序的问题。

* 子pom中版本的问题
在子pom文件中，我们一般会指定顶级pom是谁，以及顶级pom的版本是多少，当然我们也可以去指定子pom的版本，要是你的子pom版本和顶级pom的版本一样，那就别指定了，子pom自动继承顶级pom的版本，再指定pom版本就是多次一举。

####三.命令的灵活使用

* mvn dependency:tree
这命令在解决jar包冲突的时候经常用到，我们找到发生冲突的jar包，然后做响应的仲裁

* 修改pom中的版本
mvn versions:set -DnewVersion=1.0.1
执行完这个命令后，顶级pom中的version,子pom中的version,子pom中父pom的version都会被修改成1.0.1,同时我们每个被修改过的pom文件都会生成一个pom.xml.versionsBackup
执行mvn versions:revert就会把刚才修改过的version恢复。要是不执行mvn versions:revert执行mvn versions:commit就会把刚才版本的修改提交，这样pom.xml.versionsBackup文件就会被删除，commit之后就没法再回滚了。