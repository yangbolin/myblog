title: guava中的Multimap
date: 2015-04-12 14:32:31
tags: guava
categories: 编程开发
---

####一.概述
我们在日常开发过程中都会使用Map这个数据结构，使用这个数据结构能够很方便维护一些映射关系，有时候，我们存储的结构是Key-Collection的映射关系，关于这个映射关系调用put方法的时候，我们需要先调用get方法，然后看看对应的Key是否在当前映射关系中已经存在Collection，如果不存在，我们直接忘Collection中增加一个元素，不存在我们需要手动去创建，然后在往我们手工创建的Collection中增加一个元素，最后再把这个Collection写入我们的Map中，很麻烦。对于C++ STL有过了解的人都知道，STL中有一个Multimap这个数据结构，该数据结构能方便地解决上面映射关系的问题。但是JDK本身不存在这样的数据结构。。。好在guava这个开源的二方库中自己定义了这个容器，该容器既能维护好Key-Collection这个数据结构，也能很方便往这个数据结构中写数据。

<!-- more -->

####二.Multimap数据结构
![Multimap](http://bolinyoung.qiniudn.com/key-collection.png)

####三.如何使用
```java
public class MultimapTest {
    public static void main(String[] args) {
        Multimap<String, Integer> multimap = HashMultimap.create();
        multimap.put("A", 1);
        multimap.put("A", 2);
        
        System.out.println(multimap.get("A"));
    }
}
输出结果：
[1, 2]
```
关于上述代码，我们看到我们的操作很简单，并不需要开发工程师自己去创建一个Collection，然后把这个Collection在手工写入Map中去。

####四.总结
可能很多工程师自己愿意创建Collection，并且自己写到Map中去，但是这个维护起来很不好维护，其实想想自己创建Collection并且写到Map中，也是一件很麻烦的事情，从此以后对于Key-Collection这种数据结构，直接走Multimap。