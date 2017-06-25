title: 枚举在hessian序列化和反序列化中的问题
date: 2016-05-22 11:03:47
tags: JAVA
categories: 编程开发
---

####一.概述
最近在拆分一个枚举类，但是该枚举类使用在一个RPC接口上，枚举类使用在RPC接口上，必然要考虑序列化和反序列化的问题，需要确保自己对枚举的拆分不会导致序列化和反序列化的问题。

<!-- more  -->

 原来的代码为
```java
public enum xxxEnum {
  X("a11","a22"),
  Y("a111","a222"); 
  private String a1;
  private String a2;
  public String getA1(){ return a1;}
  public String getA2(){ return a2;}
 }
```
修改后的代码为
```java
public enum xxxEnum {
  X("b11","b22"),
  Y("b111","b222"); 
  private String b1;
  private String b2;
  public String getB1(){ return b1;}
  public String getB2(){ return b2;}
 }
```
 
 变更了两个成员变量的名字。这样修改反序列化会不会有问题？

####二.问题分析
要想知道序列化和反序列化会不会有问题，得先看看序列化和反序列化的源码，关于hessian的序列化和反序列化有很内容，这里就拿枚举这一个点来分析。
<font color="#336699">先看看枚举序列化的代码</font>
```java
public class EnumSerializer extends AbstractSerializer {
    private Method _name;

    public EnumSerializer(Class cl) {
        // hessian/32b[12], hessian/3ab[23]
        if (!cl.isEnum() && cl.getSuperclass().isEnum())
            cl = cl.getSuperclass();

        try {
      // 通过反射来获取枚举类的name方法
            _name = cl.getMethod("name", new Class[0]);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public void writeObject(Object obj, AbstractHessianOutput out) throws IOException {
        if (out.addRef(obj))
            return;

        Class cl = obj.getClass();

        if (!cl.isEnum() && cl.getSuperclass().isEnum())
            cl = cl.getSuperclass();

        String name = null;
        try {
      //调用枚举类的name方法来生成枚举序列化的值
            name = (String) _name.invoke(obj, (Object[]) null);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        out.writeMapBegin(cl.getName());
        out.writeString("name");
        out.writeString(name);
        out.writeMapEnd();
    }
}
```
<font color="red">关于枚举的序列化可以总结为调用枚举类的name方法来生成序列化的字符串。</font>
<font color="#336699">再看看枚举反序列化的代码</font>
```java
public class EnumDeserializer extends AbstractDeserializer {
  private Class _enumType;
  private Method _valueOf;
  
  public EnumDeserializer(Class cl)
  {
    // hessian/33b[34], hessian/3bb[78]
    if (cl.isEnum())
      _enumType = cl;
    else if (cl.getSuperclass().isEnum())
      _enumType = cl.getSuperclass();
    else
      throw new RuntimeException("Class " + cl.getName() + " is not an enum");

    try {
      // 反射获取枚举类的valueOf方法
      _valueOf = _enumType.getMethod("valueOf",
           new Class[] { Class.class, String.class });
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
  
  public Class getType()
  {
    return _enumType;
  }
  
  public Object readMap(AbstractHessianInput in)
    throws IOException
  {
    String name = null;
    
    while (! in.isEnd()) {
      String key = in.readString();

      if (key.equals("name"))
        name = in.readString();
      else
  in.readObject();
    }

    in.readMapEnd();

    Object obj = create(name);
    
    in.addRef(obj);

    return obj;
  }
  
  @Override
  public Object readObject(AbstractHessianInput in, Object []fields)
    throws IOException
  {
    String []fieldNames = (String []) fields;
    String name = null;

    for (int i = 0; i < fieldNames.length; i++) {
      if ("name".equals(fieldNames[i]))
        name = in.readString();
      else
  in.readObject();
    }

    Object obj = create(name);

    in.addRef(obj);

    return obj;
  }

  private Object create(String name)
    throws IOException
  {
    if (name == null)
      throw new IOException(_enumType.getName() + " expects name.");

    try {
      //反射调用枚举类的valueOf方法
      return _valueOf.invoke(null, _enumType, name);
    } catch (Exception e) {
      throw new IOExceptionWrapper(e);
    }
  }
}
```
<font color="red">枚举的反序列化可以总结为反射调用枚举的valueOf方法来获取最终的的枚举值。</font>

有了上面对枚举序列化反序列化源码的分析，现在我们看看相关的问题。(假定服务端做的序列化，客户端做的是反序列化，方便描述)

1.服务端枚举多了一个枚举值
假如服务端的枚举类为
```java
public enum A {
  X,
  Y,
  Z;
}
```
客户端的枚举类为
```java
public enum A {
  X,
  Y;
}
```
如果服务端返回一个A.Z给客户端，此时hessian反序列化调用枚举类的valueOf方法来获取反序列化，但是客户端的枚举类中没有Z，<font color="red">那么客户端反序列化直接跑异常。</font>

2.服务端枚举ordinal值以及枚举类成员变量值和客户端不一致
假设服务端的枚举类为
```java
public enum A {
  X("aaa"),
  Y("bbbb");//此时Y的ordinal为1，对应的value为bbb
  String value;
  A(String value) {this.value=value}
}
```
客户端的枚举类为
```java
public enum A {
  X("aaa"),
  Z("ccc"),
  Y("ddd");// 此时Y的ordinal为2对应的value为ddd
  String value;
  A(String value) {this.value=value;}
}
```
<font color="red">假如入服务端传递给客户端的是A.Y，此时客户端拿到的A.Y对应的ordinal为2，对应的value为ddd。</font>
上面这个点非常重要。
3.枚举是单例的
```java
public enum TestEnum {
    
    XX("xx");
    
    TestEnum(String value) {
        this.value = value;
    }
    
    String value;

    
    public String getValue() {
        return value;
    }

    
    public void setValue(String value) {
        this.value = value;
    }
}
public class Test {
    
    public static void main(String[] rgs) {
        TestEnum testEnum1 = TestEnum.XX;
        TestEnum testEnum2 = TestEnum.XX;
        
        testEnum1.setValue("XX");
        testEnum2.setValue("YY");
        System.out.println(testEnum1.value); // 输出 YY
        System.out.println(testEnum2.value); // 输出 YY
    }
}
```
testEnum1和testEnum2其实指向了同一个枚举引用。每次修改的都是同一个对象，所以前一个set的值被后面的set给覆盖了。

####三.总结
* 还是不要在RPC的接口中直接使用枚举类了，直接使用String就行
* 在枚举类中使用字符串时直接使用name()就行，不要再做过度封装，尽量保持枚举类的简洁
* 枚举类使用在RPC接口上的时候就一定要小心，重构的时候要注意保持ordinal
* 枚举在序列化和反序列化的时候，除了name值，其他啥都不带的
* 禁止给枚举提供set方法，没用的