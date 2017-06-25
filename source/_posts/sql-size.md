title: sql语句的大小
date: 2014-06-17 21:52:39
tags: sql
categories: 数据库
---

今天写了下面的sql语句
```sql
select XX,YY from tableXX where XX in ('aa','bb','cc')
```

<!--more-->

这个sql不是在mysql或者oracle数据库上执行，而是在一个实时计算的数据产品上执行，当in的范围超过400后，发现执行失败，咨询相关同学后，原来这个实时计算的产品中对in的范围大小有限制，缺省的限制是400。

我们平时都在使用mysql，那么mysql本身对in的范围大小是否有限制呢？mysql虽然没有对in的范围直接限制，但是限制了sql语句的长度，我们可以在不同的数据库中使用下面的命令来查看当前数据库server能接受的数据包大小
```sql
show variables like 'max_allowed_packet%'
```
sql也是server端需要接收的数据，这个配置如果是Linux系统的化，可以查看my.cnf配置文件中配置的值。

mysql不管你使用InnoDB还是MYISAM,sql的大小都有限制，和使用的数据库引擎没有关系，InnoDB支持事务，MYISAM不支持事务。