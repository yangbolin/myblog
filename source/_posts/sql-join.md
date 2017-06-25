title: SQL的JOIN总结
date: 2014-06-03 19:42:36
tags: sql
categories: 数据库
---

####一.概述
关系型数据库最大的优点就是关联查询，既所谓的JOIN，不像HBase这种Nosql的数据库，对于表和表的JOIN不怎么支持，关于SQL中的JOIN比较多，也不太好记忆，为了在后续开发中对SQL中的JOIN灵活使用，这里总结一些SQL中的一些JOIN。

<!-- more -->

####二.假设存在的表以及相关数据
假设table_a中的数据如下
![table_a](http://bolinyoung.qiniudn.com/table_a.png)
假设TableB中的数据如下
![table_b](http://bolinyoung.qiniudn.com/table_b.png)

table_a和table_b中有两行的name是一样的，table_a的3,4行和table_b的1,2行
####三.各种JOIN的示例
```sql
SELECT * FROM table_a A  LEFT JOIN table_b B ON A.name = B.name
```
![执行结果](http://bolinyoung.qiniudn.com/sql1.png)

上面SQL执行的时候以左表中的数据为基准，左表A中的数据都会出现在最终的结果中，但是右表A中的数据只有匹配到的数据参会出现在左表中。

```sql
SELECT * FROM table_a A  LEFT JOIN table_b B ON A.name = B.name WHERE B.name IS NULL
```
![执行结果](http://bolinyoung.qiniudn.com/sql2.png)
> 注意:
> LEFT OUTER JOIN 和 LEFT JOIN 效果一样

上面的执行结果表示只出现在左表A中的数据。

```sql
SELECT * FROM table_a A FULL OUTER JOIN table_b B ON A.name = B.name 
```
上述sql的执行结果如下

id      | name      | id        | name      |
--------|-----------|-----------|-----------|
1       | xyza      | null      |   null    |
3       | xyzb      | null      |   null    |
5       | xyzc      | 1         |   xyzc    |
7       | xyzd      | 3         |   xyzd    |
null    | null      | 5         |   xyze    |
null    | null      | 7         |   xyzf    |

注意mysql不支持FULL OUTER JOIN,这里的执行结果其实就是AUB

```sql
SELECT * FROM table_a A FULL OUTER JOIN table_b B ON A.name = B.name WHERE A.name IS NULL OR B.name IS NULL
```
上述sql的执行结果如下

id      | name      | id        | name      |
--------|-----------|-----------|-----------|
1       | xyza      | null      |   null    |
3       | xyzb      | null      |   null    |
null    | null      | 5         |   xyze    |
null    | null      | 7         |   xyzf    |

A和B的交集不会出现上述结果中。

```sql
SELECT * FROM table_a A RIGHT JOIN table_b B ON A.name = B.name WHERE A.name IS NULL
```
![执行结果](http://bolinyoung.qiniudn.com/sql3.png)
上述SQL的执行结果包含只出现在B中的记录

```sql
SELECT * FROM table_a A RIGHT JOIN table_b B ON A.name = B.name 
```
![执行结果](http://bolinyoung.qiniudn.com/sql4.png)

> 注意:
> RIGHT OUTER JOIN 和 RIGHT JOIN 效果一样

在执行上述SQL的时候，会以B表的数据为基准，A表匹配不到就输出null,输出结果中一定包含出现在B表中的记录。


```sql
SELECT * FROM table_a A INNER JOIN table_b B ON A.name = B.name 
```
![执行结果](http://bolinyoung.qiniudn.com/sql5.png)
上述执行结果类似计算A和B的交集。

####四.各种JOIN的总结
![各种JOIN的总结](http://bolinyoung.qiniudn.com/SQL-Join.jpg)
