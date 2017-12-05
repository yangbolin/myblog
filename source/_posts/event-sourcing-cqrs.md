title: 初识EventSourcing和CQRS
date: 2017-06-25 22:22:07
tags: Architecture
categories: 架构设计
---
EventSourcing就是事件溯源的意思，我们平时在设计系统的时候都存储了对象的最终的状态，比如一个交易订单，它当前的状态是等待买家确认收货，这个状态是由于发生了一系列事件所导致的，那么EventSourcing的思想就是存储对象的所有变更事件，根据对象的所有变更事件追溯出对象的状态。
```
state = f(E1,E2,E3....)
```
函数f就是溯源函数，这种设计思想对于数据一致性也有保证，因为它对所有的变更都是insert，即追加记录，能够很好地避免RaceCondition。不同的对象可能需要定义不同的溯源函数，由于数据库中没有存储对象当前的状态，那么需要根据一些状态来进行遍历的时候就比较麻烦了。

CQRS是一种读写分离的架构设计，CQRS最早来自于Betrand Meyer（Eiffel语言之父，开-闭原则OCP提出者）在 Object-Oriented Software Construction 这本书中提到的一种 命令查询分离 (Command Query Separation,CQS) 的概念。其基本思想在于，任何一个对象的方法可以分为两大类：
```
命令(Command):不返回任何结果(void)，但会改变对象的状态。
查询(Query):返回结果，但是不会改变对象的状态，对系统没有副作用。
```
先贴一张CQRS的架构图
![CRQS](http://7fvbqj.com1.z0.glb.clouddn.com/CQRS.jpg)
一个Command只能修改一个聚合根，对同一个聚合根的修改需要通过Command Bus来排队。一个聚合根的修改可能会导致多个领域模型状态上的变更，这些状态的变更全部以事件的方式存储，事件通过EventBus找到对应的Handler，然后更新DB中的数据。其实上面这个架构可以理解为为是CQRS和EventSouring的结合。

[CQRS参考资料](https://martinfowler.com/bliki/CQRS.html)
