title: 如何编写maven的helloworld插件
date: 2014-06-04 20:32:22
tags: maven
categories: 开发工具
---

####一.概述
maven提供了一套插件机制，我们只要按照maven定义的规范编写一个插件，maven就能运行我们的插件，在平时开发的过程中我们经常会开发一些maven的插件工具，这写插件工具可以实现类冲突检测，工程部署，工程打包...等等。这里就大致记录一下如何编写一个maven的Hello Word插件，就是说在maven编译的过程中输出Hello Word。

<!-- more -->

####二.如何编写maven插件
1.执行下面的maven命令,生成一个maven的插件工程
```
mvn archetype:create -DgroupId=com.alibaba.maven -DartifactId=maven-hello-plugin -DarchetypeArtifactId=maven-archetype-mojo
```
-DgroupId和-DartifactId就不用多解释了，插件本身也是一个二方库，二方库需要坐标，这样别人才能依赖。
-DarchetypeArtifactId=maven-archetype-mojo 创建maven插件工程的时候必须指定这个参数，参数值也需要写成上面这样。

2.进入插件工程所在的目录,并把插件工程变成一个eclipse可识别的工程，然后导入eclipse进行开发。
```
cd maven-hello-plugin
mvn eclipse:clean eclipse:eclipse
```

3.在eclipse中开发，编写下面的类
```java
import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;

/**
 * @goal helloworld
 */
public class MyMojo extends AbstractMojo {

    /**
     * @parameter expression="${helloworld.words}" default-value="Hello World!"
     */
    private String words;

    public void execute() throws MojoExecutionException {
        getLog().info(words);
    }
}
```
4.在插件的目录下面执行
```
mvn clean install
```
把这个插件install到本地的maven仓库中
5.插件的运行
```
mvn com.alibaba.maven:maven-hello-plugin:1.0-SNAPSHOT:helloworld
```
输出: Hello World
```
mvn com.alibaba.maven:maven-hello-plugin:1.0-SNAPSHOT:helloworld -Dhelloworld.words="welcome!"
```
输出: welcome

####三.注意点
* maven2中插件参数的注入都是通过javadoc的，使用的Plexus
* 每个maven插件都是一个MOJO,我们自己的MOJO必须extends AbstractMojo,同时我们需要重写execute方法
* getLog()实际上是maven编译时信息输出的地方
* 对于maven这种插件的设计理念可以在平时开发的过程中借鉴