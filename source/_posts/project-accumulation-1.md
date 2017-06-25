title: 项目中的点点滴滴
date: 2014-10-19 16:21:50
tags: Experience
categories: 编程开发
---
####一.概述
最近一直在忙项目中的事情，不过在整个开发过程中自己也有一些体会，写写这些体会，就当是自己的经验了，这里不会涉及到太深的原理，只是记录一些技巧以及一些值得注意的地方，形成一些编程的思想，理念，可以更好地运用到后续的项目之中。

<!-- more -->

####二.关于Spring定时任务
Spring定时任务在项目中经常会被使用，我们经常通过限定IP的方式保证我们自己设计的定时任务只会在一台机器上启动，关于Spring定时任务在项目中需要注意一下配置，有时候会漏写一些配置，其实没有必要死记这些配置怎么写，按照正常的思路，你需要指定要运行的任务是什么，其次，既然是定时任务，那就需要指定定时，也就是什么时候运行，最后，任务以及任务要运行的时间都有了，那么如何调度呢？也就是说运行时刻到了，谁来调度这个任务。显然配置Spring定时任务需要把握三个要素，就是上面这三个点，下面我们写一个简单的配置例子来说明一下这三个点
```xml
<!-- 1.指定要运行的任务 -->
<bean id="autoAuditFailedTaskJobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
   <property name="concurrent">
        <value>false</value>
   </property>
   <property name="targetObject">
       <ref bean="autoAuditFailedTask" />
   </property>
   <property name="targetMethod">
       <value>execute</value>
   </property>
</bean>
 <!-- 2.配置触发器 -->
<bean id="autoAuditFailedTaskCronTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
    <!-- 这里不可以直接在属性jobDetail中引用taskJob，因为他要求的是一个jobDetail类型的对象，所以我们得通过MethodInvokingJobDetailFactoryBean来转一下 -->
   <property name="jobDetail">
       <ref bean="autoAuditFailedTaskJobDetail" />
   </property>
   <!-- 每天晚上23:00点触发  -->
   <property name="cronExpression">
       <value>0 0 23 * * ?</value>
   </property>
</bean>

<!-- 3. 添加调度器 -->
<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
   <property name="triggers">
       <list>
          <ref local="autoAuditFailedTaskCronTrigger" />
       </list>
   </property>
</bean>
```
在配置的时候注意这三个点，缺一不可。

####三.关于NPE
我们在写代码的时候经常会考虑NPE的问题，对于参数经常会做NPE校验，在对每一个参数做NPE校验的时候一定要想一下该参数是否一定不为NULL，不要一味地追求NPE。

####四.关于时间的加减以及格式化
经常有这样的需求，需要把当前时间向前或者向后推算N天，以及对时间做格式化
```java
Calendar calendar = Calendar.getInstance();
calendar.setTime(new Date());
// n可以为正数，也可以为负数
calendar.add(Calendar.DAY_OF_MONTH, n);
// 时间格式化
DateFormat format = new SimpleDateFormat("MM/dd");
format.format(calendar.getTime());
```
####五.关于xstream
xstream可以实现XML和Object之间的相互转换
```xml
<dependency>
  <groupId>com.thoughtworks.xstream</groupId>
  <artifactId>xstream</artifactId>
  <version>1.4.7</version>
</dependency>
```
```java
public class Person {

    private int     age;
    private SexEnum sex;
    private String  name;

    public int getAge() {
        return age;
    }

    public SexEnum getSex() {
        return sex;
    }

    public String getName() {
        return name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setSex(SexEnum sex) {
        this.sex = sex;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
```java
public enum SexEnum {
    MALE("M"), FEMALE("F");

    String value;

    SexEnum(String value){
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public static SexEnum parse(String value) {
        for (SexEnum sexEnum : SexEnum.values()) {
            if (value.equals(sexEnum.getValue())) {
                return sexEnum;
            }
        }
        return null;
    }
}
```
```java
public class XStreamTest {
    public static void main(String[] args) {
        // Object TO XML
        List<Person> personList = new ArrayList<Person>();
        Person person1 =  new Person();
        person1.setAge(22);
        person1.setName("nuaa");
        person1.setSex(SexEnum.MALE);
        personList.add(person1);
        
        Person person2 =  new Person();
        person2.setAge(22);
        person2.setName("buaa");
        person2.setSex(SexEnum.MALE);
        personList.add(person2);
        
        XStream xstream = new XStream();
        // 使用别名
        xstream.alias("person", Person.class);
        xstream.alias("personList", List.class);
        // 注册枚举转换器
        xstream.registerConverter(new AbstractSingleValueConverter(){
            @Override
            public boolean canConvert(@SuppressWarnings("rawtypes") Class type) {
                if (type.equals(SexEnum.class)) {
                    return true;
                }
                return false;
            }
            // String转换成对象
            @Override
            public Object fromString(String str) {
                return SexEnum.parse(str);
            }

            // 对象转换成String
            @Override
            public String toString(Object obj) {
                SexEnum sexEnum = (SexEnum)obj;
                return sexEnum.getValue();
            }
        });
        
        String xml = xstream.toXML(personList);
        System.out.println(xml);

        // XML TO Object
        @SuppressWarnings("unchecked")
        List<Person> fromXMLList = (List<Person>)xstream.fromXML(xml);
        System.out.println(fromXMLList);
        for(Person person : fromXMLList) {
            System.out.println("#############################");
            System.out.println("age: " + person.getAge());
            System.out.println("sex: " + person.getSex());
            System.out.println("name: " + person.getName());
        }
    }
}
```
使用xtream可以避免自己解析XML文件。

####六.关于fastjson
fastjson把对象转化成json字符串时会出现$ref，这是由于对象之间有循环引用导致，这有可能导致堆栈溢出，我们可以使用
```java
JSON.toJSONString(object,  SerializerFeature.DisableCircularReferenceDetect)
```
来避免json串中出现$ref。