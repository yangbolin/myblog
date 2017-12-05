title: 两种分布式系统中保持数据一致性的设计方案
date: 2017-06-25 17:40:13
tags: CAP
categories: 分布式
---
CAP是分布式系统中非常重要的理论依据，任何分布式系统设计的时候都要考虑一下CAP。
```
C(一致性)：所有节点上的数据时刻保持同步
A(可用性)：每个请求都能得到一个响应，不论成功还是失败
P(分区容错)：系统能够持续提供服务，即使系统内部有消息丢失
```
CAP只能满足两条，现在大多数业务系统都是AP without C。
在平时的业务开发中，业务系统A更新完自己的数据后，还需要业务系统B更新相关的数据，要么系统A发消息给系统B，系统B受到消息后更新自己的数据，要么系统A通过RPC接口更改系统B的数据。那么如何保证系统A,B的数据一致性呢？如果要强一致的话，就需要分布式事务，但是分布式事务本身有一定的开发成本，同时也会影响系统的稳定性。
不使用分布式事务如何来保证一致性呢？只能放弃强一致性，变成最终一致性。一般会有两种方案，一种是通过消息中间件来实现，一种是利用数据库事务加上异步轮询来处理。

【通过消息中间件来实现】
![消息中间件](http://7fvbqj.com1.z0.glb.clouddn.com/%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6%E4%B8%80%E8%87%B4%E6%80%A7.png)
这种方案需要相应的消息中间件来支持。
【利用数据库事务+异步轮询】
![事务轮询](http://7fvbqj.com1.z0.glb.clouddn.com/%E4%BA%8B%E5%8A%A1+%E8%BD%AE%E8%AF%A2.png)
业务系统A更新完自己的数据后，创建一个事件，创建事件和更新自己业务数据放在一个事务里面，这样业务系统A数据更新和事件创建会一起完成，剩下的就是保证业务事件A一定要处理，要么通过RPC调用业务系统B的接口，要么发消息给业务系统B。

