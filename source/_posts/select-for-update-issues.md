title: mysql中select for update锁的问题
date: 2017-12-05 19:10:19
tags: mysql
categories: 编程开发
---
在日常关于资金业务的开发过程中，涉及到数据库的操作时我们经常按照一查二锁三写的套路，可能在加锁后还需要查询一些数据，因为加锁之前的数据可能已经发生变化了，这里的锁我们一般会采用数据库的select for update锁，使用这个本身也没有问题，在对数据进行操作的时候先加锁，避免其他线程修改后，当前线程看到的数据是错误的。但是在高并发的情况下面，这个select for update锁的等待时间太久导致很多线程都在等待这个锁，这样整个系统的吞吐量严重下降。举个栗子在电商业务中有个系统自动确认收货，确认收货后会更改一些用户的数据，如果同卖家在双11当天同时卖出了上万件商品，生成了上万笔订单，这上万笔订单系统自动确认收货的时间点都是一样的，到点后会爆发非常大的并发流量，如果先select for update的话，很多线程就会排队等待，系统吞吐量下降，数据库链接数耗尽，出现故障。

在高并发系统中，如果出现了获取不到锁的情况，应该尽快返回让上游系统发起重试，而不是一直等直到可以获取到锁，一直的等待绝对会把系统拖死，这种case目前已经经历了好多了。
