title: Java 注解总结
date: 2014-05-18 16:51:38
tags: JAVA
categories: 编程语言
---

####一.概述
在写JAVA代码的时候，我们经常会使用注解来简化代码，这些注解是需要我们自己定义的，当然JAVA本身也提供了一些标准注解。同时我们也建议大家在自己的代码中定义自己的注解，为了能够在代码中灵活使用注解，这里总结一下注解相关的东西。

<!-- more -->

####二.三种标准注解和四种元注解
* 三种标准注解
@Override 标识当前方法必须在父类或者接口中存在，不然会出现编译错误
@Deprecated 标识当前类，方法或者字段已经被废弃，不建议再使用
@SuppressWarnings 用来关闭一些警告，比如可以使用@SuppressWarnings{"unchecked"}来关闭类型转换的警告

* 四种元注解
元注解是用来定义注解的。
@Target
标识该注解可以用在什么地方，可能的ElementType如下
CONSTRUCTOR:构造函数声明
FIELD:字段声明
LOCAL_VARIABLE:局部变量声明
METHOD:方法声明
PARAMETER:参数声明
ANNOTATION_TYPE:注解类型声明
TYPE:类，接口或者枚举的声明
PACKAGE:包声明
@Retention
标识需要在什么级别保存注解信息，可能的RetentionPolicy如下
SOURCE:注解被编译器丢弃
CLASS:注解在class文件中可用，但是被JVM虚拟机丢弃
RUNTIME:注解在JVM虚拟机中保留，因此可以通过反射获取到注解信息
@Documented
此注解将会包含在javadoc中
@Inherited
允许子类继承父类中的注解

####三.注解例子

* 简单注解

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import com.taobao.Constraints;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraints {
    boolean primaryKey() default false;
    boolean allowNull() default true;
    boolean unique() default false;
}
```

这里看到有三个方法声明，其实这三个方法都是注解的元素，不是什么方法。因此我们使用注解的时候需要指定一下元素的值，比如
```java
@Constraints(primaryKey = true, allowNull = false, unique = true)
```
* 注解中的注解

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import com.taobao.Constraints;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)

public @interface DBColumn {
    int value() default 0;
    Constraints constraints() default @Constraints;
}
```

在DBColumn中我们使用了另外一个注解Constraints，即所谓的注解中使用注解。在使用DBColumn这个注解的时候，我们可以这样写
```java
@DBColumn(value = 88, constraints=(@Constraints(primaryKey = true, allowNull = false, unique = true)))
```

在定义注解的时候，为元素提供defaultValue是一个好习惯。

####四.注解的获取
在JAVA代码中我们一般使用反射来获取注解
```java
// 获取类上面的注解
Class.getAnnotations()
// 获取字段上面的注解
Field.getAnnotations()
// 获取方法上面的注解
Method.getAnnotations()
```
