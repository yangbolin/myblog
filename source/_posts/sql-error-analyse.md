title: 一个sql问题的排查
date: 2014-04-21 19:26:01
tags: sql
categories: 数据库
---

####一.现象
线上一台任务机对一张mysql表进行全表扫描，当扫描过程中有一行记录读出来做数据转换，发现有很多行被多次读出来，并且多次尝试去做数据转换，同时数据转换也就失败了，因此也就打出了error级别的日志。

<!-- more -->

####二.分析
由于这个任务是在遍历表，因此我们直接找到相应的sql语句看看，下面就是找到的sql语句
```xml
<![CDATA[
	select xxx,xxxx
	from xxxxyyyyy
	where gmt_create >=  #beginTime#
    and gmt_create <  #endTime#
    and id >= #idBegin#
    limit #pageSize#     
]]>
```
上面的sql在一个循环中会不断被调用，下面我们先来看看调用这个sql的java代码
```java
// 找到数据库中最大的主键ID
long maxId = findCompanyMaxId(TABLE_NAME);
// 循环次数
long loopTime = 1;
do {
    // 计算开始id
    long idBegin = maxId - loopTime * SCOPE_OF_EACH_LOOP + 1;
    long idEnd = maxId - (loopTime - 1) * SCOPE_OF_EACH_LOOP;

    List<Xxxx> currentList = null;

    if (idBegin > 0) {
        // 调用上面的sql语句获取数据,beginTime和endTime在这个方法中设置，因为这个两个值可以系统当前时间推算出来
        currentList = xxDAO.getListFromMaxId(days, TABLE_NAME, TABLE_PK, idBegin, idEnd);
    } else {
        // 退出循环
        break;
    }
    
    if (currentList == null || currentList.size() == 0) {
        // 退出循环
        break;
    }
} while (true);
```

上面的代码很简单，先找出数据库中表主键的最大ID，select max(id) from ...即可，然后每次把maxId向前推进SCOPE_OF_EACH_LOOP,这样就能不断遍历这张表了。我们假设第一次获取到的maxId为9000,每次向前推进1000，pageSize=1000,这样第一调用sql时的idBegin=9000 - 1000 + 1 = 80001,这时候我们取出满足条件的limit 1000条记录，但是最终结果可能不足1000条，因为有beginTime和endTime的限制，好了第一次调用这个sql没啥问题的，后面我们考虑一次sql的调用，idBegin = 9000 - 2 * 1000 + 1 = 7001,这时候我们指定的idBegin=7001,limit大小为1000，其实这个sql的本质期望是扫描[7001,8000]之间的记录，但是[7001,8000]之间符合条件的记录又不足1000条，这样就会从[80001,9000]之间再去取一些行来补足1000行，谁让你写了1000呢？但是[8001,9000]之间的记录明显已经被扫描过了，以此类推数据的重复扫描就会发生了，悲剧由此而产生，只能说这种sql写的太没水平了。

####三.解决办法
修改sql，删掉limit，增加idEnd的限制，修改后的sql如下:
```xml
<![CDATA[
	select xxx,xxxx
	from xxxxyyyyy
	where gmt_create >=  #beginTime#
    and gmt_create <  #endTime#
    and id >= #idBegin#
    and id <= #idEnd#
]]>
```
注意:

* 当sql语句中出现了limit并且有比较关系的时候，注意考虑sql语句会不会出现重复扫描数据的逻辑。
