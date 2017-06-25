title: 如何动态修改JVM的字节码
date: 2015-10-13 19:13:30
tags: JVM
categories: JVM学习
---
####一.概述
最近在做一个需求的时候，需要在JVM启动好之后，能够动态的修改JVM已经加载的一个类的一个方法，把这个方法的返回值直接改成返回true。

<!-- more -->

上述需求概括为动态修改JVM字节码，我们需要借助修改字节码的工具，同时也要让启动中的JVM能感知到我们的修改，这个需要借助java的instrument。下面我们就来看一下具体的实现。
####二.实现
#####1.编写修改字节码的agent
在JVM已经加载的类中找到要修改的类，然后使用javassist从磁盘上读取到要修改类的字节码，修改指定方法的返回值后，让JVM重新再加载一下我们刚才修改的这个类，具体代码如下：
agent的代码
```java
public class XxxmockAgent {
    // 指定我们要修改字节码的类的全限定名
    private static final String CLASS_NAME = "xxxCommonBO";
    @SuppressWarnings("rawtypes")
    public static void agentmain(String agentArgs, Instrumentation inst) throws UnmodifiableClassException{
        System.out.println("loadagent after main...");
        //获取当前JVM已经加载过的所有类
        Class[] classes =  inst.getAllLoadedClasses();
        for (Class clazz : classes) {
	    //找到需要修改的类
            if(clazz.getName().equals(CLASS_NAME)) {
                System.out.println("find class " + CLASS_NAME);
                //按照要求字节吗
                inst.addTransformer(new XxxCommonTransformer(), true);
                //让JVM重新加载修改过字节码的类
                inst.retransformClasses(clazz);
            }
        }
        System.out.println("loadagent after main sucess...");
    }
}
```
修改字节码的代码
```java
public class XxxCommonTransformer implements ClassFileTransformer {

    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain,
                            byte[] classfileBuffer) throws IllegalClassFormatException {
        String compareClass = className.replace('/', '.');
        System.out.println("transformer..." + compareClass);
        try {
	    //构建javassist需要ClassPool
            ClassPool classPool = ClassPool.getDefault();
            //把要修改的类的classpath加入到javassist的ClassPool中
            classPool.appendClassPath("/xxx/WEB-INF/lib/*");
            //从磁盘上读取要修改类的字节码，并且转换成javassit中的CtClass模型
            CtClass ctClass = classPool.get(compareClass);
            //获取需要修改的字节码的方法
            CtMethod ctMethod = ctClass.getDeclaredMethod("isFromA");
            //修改方法体
            ctMethod.setBody("return true;");
            //写入修改后的字节码
            ctClass.writeFile();
            return ctClass.toBytecode();
        } catch (Exception e) {
	    e.printStackTrace();
        }
        return null;
    }
}
```
#####2.编写attach到JVM的client
```java
public class Client {
    public static void main(String[] args) {
        if (args.length != 2) {
            System.out.println("[usage:java -jar client-1.0.0.jar pid path] and args.lenght="+args.length);
            return;
        }
        // 第0个参数是要attach的JVM进程ID
        String pid = args[0];
        // 第1个参数是agent JAR包所在的路径
        String agentPath =args[1];
        System.out.println("pid:" + pid);
        System.out.println("agentPath:" + agentPath);
        try {
            attach(pid, agentPath);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public static void attach(String pid, String agentPath) throws Exception {
        try {            
            //attach到远程JVM上去
            VirtualMachine  vm = VirtualMachine.attach(pid);
            //加载agent
            vm.loadAgent(agentPath);
        } catch (RuntimeException re) {
            throw re;
        } catch (IOException ioexp) {
            throw ioexp;
        } catch (Exception exp) {
            exp.printStackTrace();
            throw exp;
        }
    }
}
```
####三.注意点
1.在编写agent的时候我们需要指定要修改的类所在的classpath，此时的类加载器是AppClassLoader，如果你要attach的JVM进程是用jetty&&tomcat等容器启动起来的，必须要指定要修改的类所在的classpath。
2.在agent中我们使用到javassit这个开源的操作字节码的二方库，加载javassit中类的类加载器也是AppClassLoader，同样如果你的JVM进程是用jetty或者tomcat启动的话，而且你的应用中已经包含了javassit这个二方库，AppClassLoader也加载不了，会出现ClassPool加载失败的异常，此时需要我们显式地把javassit包含到agent中去，如果你的agent是使用maven构建的话，你可以使用maven-shade-plugin这个maven插件，该插件既能把依赖的jar聚合起来，也能在jar包中自动生成MANIFEST.MF。
3.在编写client的时候我们用到了VirtualMachine这个类，这个类在tools.jar中，这个jar是jdk自己携带的。如果你是用maven来构建你的client的话，你可以用下面方式把tools.jar自动引入到你的project中。
```xml
<dependency>					
	<groupId>com.sun</groupId>
	<artifactId>tools</artifactId>
	<version>1.6</version>
	<scope>system</scope>
	<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
</dependency>
```
4.在运行client的方式，我们通过java -jar来运行，此时需要tools.jar，如果你不想把tools.jar打包的client中，你需要在运行client JAR的时候带上tools.jar.
```java
java -Xbootclasspath/a:tools.jar  -jar /home/xxx/client-1.0.0.jar 22410 /home/xxx/agent-1.0.0.jar 
```
22410是进程PID

