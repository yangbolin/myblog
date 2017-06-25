title: mysql相关的注意点
date: 2017-02-17 19:20:33
tags: mysql
categories: 数据库
---

最近一直忙于业务系统的开发，在开发过程中少不了sql语句的使用，我们在分页遍历的时候经常需要排序，就会编写如下的sql
```sql
select x, y, z from tableX where status=#status#
order by apply_time
limit 0,100
```
这个分页sql看上去没有什么问题，但是在执行过程中发现有数据被重复遍历出来，然后仔细排查发现上面sql中的order by apply_time是存在问题的，因为多条记录如果apply_time是一样的话，会被重复扫描到，排序字段一样，同一条记录可能会出现在不同的页中。在编写mysql分页遍历语句时，一定要注意排序字段的选择，主要重复筛选的问题。

现在整个业界对于分布式锁暂时还没有一些好的做法，我自己习惯使用mysql的select for update来实现锁，但是在使用select for update时，如果where语句中不带主键id的话，该sql可能会锁表，因为mysql发现where语句匹配到的记录数目比较多，对多行加锁的代价比对单表加锁的代价高，因此mysql会对表加锁。使用select for update最安全的做法是带上主键ID。如果要加锁多行，写个for循环对多条记录按照固定的顺序依次执行select for update。

