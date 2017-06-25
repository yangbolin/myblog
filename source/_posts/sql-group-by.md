title: 谈谈sql语句中的group by
date: 2014-04-13 20:39:50
tags: sql
categories: 数据库
---

####一.概述
我们经常使用sql语句来从数据库中查询一批数据出来，有时候我们需要把查询出来的数据分组，然后进行其他操作，比如统计，在实现这个需求的时候我们就会用到group by，从字面来解释按照XX来分组，到底按照什么来分组，需要我们在写sql语句的时候显式地指定，可以指定一个字段，也可以指定多个字段，通常group by和sql的函数结合起来使用。

<!-- more -->

####二.语法
```sql 
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE condition
GROUP BY column_name1...column_nameN
HAVING condition
```
注意:

* aggregate_function表示sql的一些内聚函数，例如count，sum等
* WHERE不是必须的
* GROUP BY后面可以指定多个字段，最后每个分组中，这几个字段的值都是一样的
* HAVING 可以选出符合某些条件的组，放弃一些不符合条件的组，在HAVING子句中我们也可以使用一些sql的内聚函数

####三.实例
假如我们有如下的表member_trade_record
trade_type表示交易类型，线上or线下

id        | member_id | trade_money | trade_type |
--------- | --------- | --------- | --------- |
1         | nuaa_2012 | 1200        | ONLINE     |
2         | nuaa_2013 | 1201        | OFFLINE    |
3         | nuaa_2014 | 1202        | ONLINE     |
4         | nuaa_2012 | 200         | OFFLINE    |
5         | nuaa_2013 | 300         | OFFLINE    |

现在需要统计每个人的交易总额，可以使用如下语句
```sql
select memer_id, sum(trade_money) from member_trade_record group by member_id
```
输出如下：

member_id | sum(trade_money) |
--------- | --------- |
nuaa_2012 | 1400 |
nuaa_2013 | 1501 |
nuaa_2014 | 1202 |

现在需要统计每个人线上线下的交易总额分别是多少,这时候就需要制定多个字段了
```sql
select member_id, trade_type, sum(trade_money) from  member_trade_record group by member_id, trade_type
```
输出如下：

member_id | trade_type | sum(trade_money) |
--------- | --------- | --------- | 
nuaa2012  | ONLINE    | 1200      |
nuaa2012  | OFFLINE   | 200       |
nuaa2013  | OFFLINE   | 1501      |
nuaa2014  | ONLINE    | 1202      |

现在我们需要选出交易总额大于等于1500的会员
```sql
select member_id, sum(trade_money) from member_trade_record group by member_id having sum(trade_money) >= 1500
```
输出如下:

member_id | sum(trade_money) |
--------- | --------- |
nuaa_2013 | 1501 |

####四.总结
我们不使用group by也可以实现上面的需求，但是使用group by我们能更快更优雅的实现上面的需求，要是想对group by后的组进行更细粒度的筛选，就必须使用having子句来实现了。

