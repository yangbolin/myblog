title: JAVA单元测试mock框架
date: 2016-07-17 10:50:10
tags: JAVA
categories: 编程开发
---
####一.概述
最近在做代码重构，发现系统中的UT很少，重构没有UT的话，全部得人工测试，逻辑覆盖不一定全部能覆盖到，因此UT还是很有必要存在的。在写UT的时候，mock是必须要有的，但是现在适用于java代码做单元测试的mock框架很多，我们该如何选择？

<!-- more -->

在做选择之前，我们先看看如何使用每个mock框架，再做决定。

####二.mock框架
为了方便描述，我们先写一个简单的测试类
```java
public class MockClazz {

    public String run(String name) {
        System.out.println(name);
        return name + " begin run...";
    }

    public static String sleep (String name) {
        System.out.println(name);
        return name + " begin sleep...";
    }

    private String eat(String name) {
        System.out.println(name);
        return name + " begin eat...";
    }

    public String getEatInfo(String name) {
        return eat(name);
    }

    public final String create(String name) {
        return name;
    }
}
```
####1.easymock
easymock是比较早的一个mock框架，做一次mock需要先创建一个mock对象，然后录制mock代码，把mock对象切换到播放状态，执行单元测试，最后再验证mock对象是否按照录制的mock行为执行。
引入easymock的依赖
```xml
<dependency>
    <groupId>org.easymock</groupId>
    <artifactId>easymock</artifactId>
    <version>3.4</version>
</dependency>
```
如何编写mock代码，分为五个步骤
```java
// ① 创建mock对象
MockClazz mockClazz = EasyMock.createMock(MockClazz.class);
// ② 录制mock对象的预期行为和输出
EasyMock.expect(mockClazz.run(EasyMock.anyString())).andReturn("mocked string for run");
// ③ 将mock对象切换到播放状态
EasyMock.replay(mockClazz);
// ④ 调用mock对象方法进行测试
String actualString = mockClazz.run("name");
Assert.assertEquals("mocked string for run", actualString);
// ⑤ 对mock对象的行为进行验证,验证mock的对象是否按照录制的行为发生
EasyMock.verify(mockClazz);
```
这样我们就使用easymock完成了一个对象的mock测试。
在上面的例子中，我们只mock了run这个方法，那没有被mock的方法，在调用的时候会出现什么问题？
```java
String eatInfo = mockClazz.getEatInfo("aa");
```
上述代码执行的时候出现下面的异常
><font color="red">java.lang.AssertionError: 
  Unexpected method call MockClazz.getEatInfo("aa"):
  at org.easymock.internal.MockInvocationHandler.invoke(MockInvocationHandler.java:44)
	.....
</font>
	
可见easymock如果其中一个方法没有被mock但是被调用了，就会抛异常。easymock不支持private,final,static等方法的mock

####2.mockito
mockito是在easymock之后出现的，相对于easymock来说，mockito少了对象状态切换这一步骤。
引入mockito的依赖
```xml
<dependency>
     <groupId>org.mockito</groupId>
     <artifactId>mockito-all</artifactId>
     <version>2.0.2-beta</version>
</dependency>
```
如何编写mockito的mock代码，分四个步骤
```java
// ① 创建mock对象
MockClazz mockClazz = Mockito.mock(MockClazz.class);
// ② 录制mock代码
Mockito.when(mockClazz.run("A")).thenReturn("B");
// ③ 执行单元测试
String actual = mockClazz.run("A");
Assert.assertEquals(actual,"B");
// ④ 校验mock对象的行为是否按照mock执行
Mockito.verify(mockClazz).run("A");
```
和easymock相比，mockito少了一个环节，就是把对象切换到播放状态。
上面的代码中我们mock了run方法，但是getEatInfo方法没有被mock,调用这个方法会出现什么问题
```java
String eatInfo = mockClazz.getEatInfo("aa");
System.out.println(eatInfo);
```
此时返回null，按照mockito的官方文档，没有被mock的方法返回默认值，具体可以看mockito的官方文档。
那么mockito如何保证不被mock的代码按照原来的逻辑输出呢？
<font color="red">通过doCallRealMethodl来实现</font>
```java
MockClazz mockClazz = Mockito.mock(MockClazz.class);
Mockito.doCallRealMethod().when(mockClazz).run("A");
String actual = mockClazz.run("A");
System.out.println(actual); // A begin run... 原样执行

System.out.println(mockClazz.run("B")); // null 返回默认值
```
上面的代码显示指定了通过run("A")的时候调用原来的代码执行，输出A返回A begin run...
<font color="red">通过spy来实现</font>
```java
MockClazz mockClazz = Mockito.spy(new MockClazz()); // 注意这里需要new一个
Mockito.when(mockClazz.run("A")).thenReturn("B");

String actual = mockClazz.run("C");
System.out.println(actual); // 输出[C begin run...],原样输出忽略mock逻辑
```
此时mockClazz.run("C")直接按照原来的代码执行，忽略mock逻辑。
在使用spy的时候需要注意一个点，看下面两段代码
```java
MockClazz mockClazz = Mockito.spy(new MockClazz());
Mockito.when(mockClazz.run("A")).thenReturn("B");
System.out.println(mockClazz.run("A")); // 实际执行run的代码,只是修改返回值（先输出A,再返回B begin run...)
```
这段代码只是修改了返回值，实际代码逻辑被执行了，也就是说这种mock逻辑只是mock了返回值，类似SpringAOP在方法返回的时候拦截一下修改了返回值。
```java
MockClazz mockClazz = Mockito.spy(new MockClazz());
Mockito.doReturn("B").when(mockClazz).run("C");
System.out.println(mockClazz.run("C")); // 根本不执行run的代码,直接返回
```
这段代码不仅该了返回值，同时也真正的代码一行也不会执行。

> 注意这两种写法的微妙区别哦
> Mockito.doReturn("B").when(mockClazz).run("C");
> Mockito.when(mockClazz.run("A")).thenReturn("B");

mockito不支持private,final,static等方法的mock。

####3.powermock
powermock实在easymock和mockito的基础上扩展而来的，easymock和mockito不能解决private,final,static等方法的mock，powermock为此提供了解决方案。powermock需要和easymock或者mockito配合起来一起使用。
引入依赖
```xml
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito</artifactId>
    <version>1.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>1.5</version>
    <scope>test</scope>
</dependency>
```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest( { MockClazz.class })
public class MockTest {

    @Test
    public void testMockStatic() {
        // 静态方法的mock
        PowerMockito.mockStatic(MockClazz.class);
        PowerMockito.when(MockClazz.sleep("A")).thenReturn("B");
        System.out.println(MockClazz.sleep("A"));
        PowerMockito.verifyStatic();
    }

    @Test
    public void testMockPrivate() throws  Exception {
        // 私有方法的mock,getEatInfo=>eat,eat是私有方法
        MockClazz mockClazz = PowerMockito.mock(MockClazz.class);
        PowerMockito.when(mockClazz, "eat", "A").thenReturn("mock");
        PowerMockito.doCallRealMethod().when(mockClazz).getEatInfo("A");
        System.out.println(mockClazz.getEatInfo("A"));
    }

    @Test
    public void testMockFinal() {
        // final方法的mock
        MockClazz mockClazz = PowerMockito.mock(MockClazz.class);
        PowerMockito.when(mockClazz.create("A")).thenReturn("B");
        System.out.println(mockClazz.create("A"));
    }
}
```
#####4.jmockit
jmockit是一个轻量级的mock框架，内部采用ASM来修改字节码。
引入依赖
```xml
<dependency>
    <groupId>com.googlecode.jmockit</groupId>
    <artifactId>jmockit</artifactId>
    <version>1.0</version>
</dependency>
```
具体mock的代码如下
```java
@RunWith(JMockit.class)
public class MockTest {

    @Mocked
    MockClazz mockClazz = new MockClazz();

    @Test
    public void testExpectations() { // 全局mock抛异常
        new Expectations() {
            {
                mockClazz.getEatInfo("A");
                returns("B");
            }
        };

        // 被mock的方法,返回mock后的值
        System.out.println(mockClazz.getEatInfo("A"));
        // 没有被mock的方法,mockit.internal.UnexpectedInvocation,jmocit对run没有进行mock
        System.out.println(mockClazz.run("A"));
    }

    @Test
    public void testNonExpectations() { //全局mock返回缺省值

        new NonStrictExpectations() {
            {
                mockClazz.getEatInfo("A");
                returns("B");
            }
        };

        // 被mock的方法,返回mock后的值
        System.out.println(mockClazz.getEatInfo("A"));
        // 没有被mock的方法,返回默认值,jmockit对run方法也进行了mock
        System.out.println(mockClazz.run("A"));
    }

    @Test
    public void testMockStatic() {
        // mock静态方法
        new NonStrictExpectations() {
            {
                MockClazz.sleep("A");
                result = "B";
            }
        };
        System.out.println(MockClazz.sleep("A"));
    }

    @Test
    public void testMockPrivate() {
        // mock静态方法
        final MockClazz obj = new MockClazz();
        new NonStrictExpectations(obj) {
            {
                // 私有方法mock
                this.invoke(obj, "eat", "A");
                returns("B");
            }
        };
        System.out.println(obj.getEatInfo("A")); // 私有方法被mock了
        System.out.println(obj.run("A")); // run方法不会被mock,走真实逻辑
    }
}
```
>  注意：
>  1.NonStrictExpectations返回缺省值针对没有mock的方法
>  2.Expectations针对没有mock的方法直接抛异常
>  3.官网上的jmockito暂时不支持私有方法的mock，google提供的高版本二方库也不支持私有方法的mock

上面例子的代码https://github.com/yangbolin/java-mock-framework

####三.总结
本文依次对比了easymock,mockito,powermock,jmockito四个java的mock框架，easymock,mockito都存在final,private,static方法没法mock的问题，powermock解决了这个问题，jmockit也没有这个问题，powermock和jmockit的区别就在于API的风格，powermock继承了easymock和mockito的风格，链式的API风格，非常清晰，jmockit有自己的API风格。

