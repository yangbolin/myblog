title: 如何使用ASMifier
date: 2014-07-27 13:53:45
tags: ASM
categories: 编程开发
---

####一.概述
ASM这个字节码处理框架，不仅能够读字节码，而且还能修改字节码，我们经常是先写好JAVA代码，然后编译(IDE或者javac)生成class文件，最后classloader再把相应的class文件加载到内存，但是有了ASM后，我们可以直接使用ASM生成class文件的字节数组，即面向字节码写JAVA代码，这样可以让JVM动态去加载一个类，但是这个类的class文件并不存在磁盘上，也就是说这个类的class文件是在内存中构建出来的。

<!-- more -->

下面我们就来看看如何在内存中构造class文件的字节数组，并且让JVM动态加载对应的class.

####二.如何使用
假如我们我们需要JVM动态加载的类的源代码如下
```
public class ASMTest {
    public void hello() {
        System.out.println("Hello ASM");
    }
}
```

此时这类并不再当前应用所对应的classpath中。当然我们可以自己去分析这个类的字节码，然后一步一步去构造这个类的字节码，但是这样比较麻烦，ASM提供了这样一个工具，可以把一个编译过的class文件转换一段java代码，运行这段java代码可以产生这个编译过的class文件对应的字节数组。

因此，我们先执行
```
javac ASMTest.java
```
生成ASMTest对应的class文件。接下来我们使用ASMifier来生成可以产生字节数组的代码
```
java -classpath asm-debug-all-4.1.jar:. org.objectweb.asm.util.ASMifier ASMTest > ASMTestDump.java
```
在执行java命令的时候我们指定了classpath,多个classpath使用冒号分隔，此时一定要保证ASMTest的所在的路径被加入到classpath中，生成的ASMTestDump类如下。

```java
public class ASMTestDump implements Opcodes {

    public static byte[] dump() throws Exception {

        ClassWriter cw = new ClassWriter(0);
        MethodVisitor mv;

        cw.visit(V1_6, ACC_PUBLIC + ACC_SUPER, "ASMTest", null, "java/lang/Object", null);
        {
            mv = cw.visitMethod(ACC_PUBLIC, "<init>", "()V", null, null);
            mv.visitCode();
            mv.visitVarInsn(ALOAD, 0);
            mv.visitMethodInsn(INVOKESPECIAL, "java/lang/Object", "<init>", "()V");
            mv.visitInsn(RETURN);
            mv.visitMaxs(1, 1);
            mv.visitEnd();
        }
        {
            mv = cw.visitMethod(ACC_PUBLIC, "hello", "()V", null, null);
            mv.visitCode();
            mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
            mv.visitLdcInsn("Hello ASM");
            mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V");
            mv.visitInsn(RETURN);
            mv.visitMaxs(2, 1);
            mv.visitEnd();
        }
        cw.visitEnd();

        return cw.toByteArray();
    }
}
```

接下来定义自己的classloader

```java
public class MyClassLoader extends ClassLoader {

    public Class defineClass(String clazz, byte[] bytes) {
        return defineClass(clazz, bytes, 0, bytes.length);
    }
}
```

使用自己的classloader加载ASMTestDump产生的字节数组,并通过反射来调用ASMTest类的Hello方法

```java
public class ClassWriterTest {

    public static void main(String[] args) {
        MyClassLoader mycl = new MyClassLoader();
        Class classInstance = null;
        try {
            classInstance =  mycl.defineClass("ASMTest", ASMTestDump.dump());
        } catch (Exception e) {
            e.printStackTrace();
        }
        
        if (classInstance != null) {
            try {
                Object oo = classInstance.newInstance();
                Method method = classInstance.getMethod("hello", null);
                method.invoke(oo, null);
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

使用ASMifier就这么简单。

####三.总结
ASMifier其实也可以认为是一个编译器，把class文件转换成java代码，这里的java代码并不是反编译后的java代码，而是一段可以生成当前class字节数组的代码，可能很多人会有以为，class文件到字节数组需要这么麻烦吗？我们直接读取class文件，然后字节转换成字节数组不就搞定了么，不用这么绕一大圈的么，关键是有时候字节码并不是存在磁盘上的，需要在内存中动态构建，如果你愿意在内存中写字节码也行。
