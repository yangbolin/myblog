title: HBase的基础类型KeyValue
date: 2014-07-20 16:22:39
tags: HBase
categories: 大数据
---

####一.概述
HBase是面向列的存储数据的，最终的存储单元都是KeyValue的结构，HBase本身也定义了一个KeyValue的类型，这是HBase数据存储的基本类型。那么KeyValue本身会存储那些信息呢？

<!-- more -->

从名字来看应该只有两个数据，一个是Key,一个是Value,的确如此，不过这里的Key是多个元素的聚合，有rowkey,列族，列名，时间戳以及key的类型，key的类型定义如下

```java
  public static enum Type {
    Minimum((byte)0),
    Put((byte)4),

    Delete((byte)8),
    DeleteColumn((byte)12),
    DeleteFamily((byte)14),

    // Maximum is used when searching; you look from maximum on down.
    Maximum((byte)255);
    ....
  }
```

####二.KeyValue结构概述
![HBase的KeyValue](http://bolinyoung.qiniudn.com/HBase-key-value.png)

HBase的KeyValue内部维护着一个字节数组，然后通过不同的偏移量来获取不同的部分，前面说过KeyValue本身就两部分，Key&&Value,因此KeyLength标识KeyValue中Key在字节数组中所占的长度，ValueLength标识Value在字节数组中所占的长度。观察上图，我们看到从RowLength到KeyType都是KeyValue这个基本类型的Key,我们来看一下这个Key中包含那些东西，RowLength即rowkey的长度，RowKey即rowkey的内容，ColumnFamilyLength即列族的长度，ColumnFamily即列族的内容，ColumnQualifier即列的名称，TimeStamp即时间戳，KeyType即Key的类型，前面已经介绍过。

我们看到从ColumnQualifier开始内容前面不在带有长度了，关于TimeStamp和KeyType很好理解，因为他们所占的字节数目是固定，时间戳是一个long型的数字，占固定、字节数目，KeyType看其定义就能知道占1个字节，此时就剩下ColumnQualifier了，列名所占的字节数目计算一下即可，KeyLength-RowLength-ColumnFamilyLength即可，其中TimeStamp以及KeyType所占的字节长度不计算到KeyLength中，虽然他们是Keyalue中key的一部分，原因就是他们的长度固定，没有必要单独表示。
下面我们看看KeyValue中计算列名所占字节数目的代码
```java
  // 字节数组中用固定长度的字节数目表示内容所占的字节数目
  public static final int KEY_INFRASTRUCTURE_SIZE = ROW_LENGTH_SIZE
      + FAMILY_LENGTH_SIZE + TIMESTAMP_TYPE_SIZE;
      
  ......
  /**
   * @return Qualifier length
   */
  public int getQualifierLength() {
    return getQualifierLength(getRowLength(),getFamilyLength());
  }

  /**
   * @return Qualifier length
   */
  public int getQualifierLength(int rlength, int flength) {
    // KeyLength-表示长度的字节数目-rowKeyLength-familyLength即列名所占的字节数目
    return getKeyLength() -
      (KEY_INFRASTRUCTURE_SIZE + rlength + flength);
  }
```