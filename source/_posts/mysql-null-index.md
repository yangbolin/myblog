title: mysql中的null
date: 2017-06-22 20:10:25
tags: mysql
categories: 数据库
---
最近在优化系统代码的时候，需要修改一下表结构，增加了一个字段D，原来表中A,B,C三字段构成了唯一性约束，现在增加了D字段，D字段是可控的，没有缺省值。同时新的唯一性约束由A,B,C,D四个字段构成，上线后发现唯一性约束失效啦。
a1,b1,c1,null插入数据库后，a1,b1,c1,null还可以插入数据库。也就是说mysql任务[a1,b1,c1,null]和[a1,b1,c1,null]是两条记录，的确是这样，在mysql中null和null是不相等的。
