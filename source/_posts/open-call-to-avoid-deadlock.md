title: 通过开放调用来避免死锁
date: 2014-10-25 21:00:45
tags: Concurrency
categories: 并发技术
---

####一.概述
在编写并发程序的时候，我们经常会在一次业务逻辑处理过程中获取多个锁，然后做一些操作，如果这多个锁的获取顺序和其他地方的获取顺序不同的话，很容易发生死锁。为了避免这种死锁，我们必须保证获取多个锁的顺序一致，或者方法调用为开放调用，即调用某个方法时不需要持有锁。

<!-- more -->

####二.示例分析
Taxi表示一个出租车对象，包含位置和目的地两个属性，Dispatcher代表一个出租车队。
```java
class Taxi {
	@GuardedBy("this")private Point location, destination;
	private final Dispatcher dispatcher;
	public Taxi(Dispatcher dispatcher) {
		this.dispatcher = dispatcher;
	}
	/** 获取出租车的位置 **/
	public synchronized Point getLocation() {
		return location;
	}
	/** 设置出租车的位置 **/
	public synchronized void setLocation(Point location) {
		this.location = location;
		if (location.equals(distination)) {
			dispatcher.notifyAvailable(this);
		}
	}
}
```

```java
class Dispatcher {
	@GuardedBy("this") private final Set<Taxi> taxis;
	@GuardedBy("this") private final Set<Taxi> availableTaxis;
	
	public Dispatcher() {
		taxis = new HashSet<Taxi>();
		availableTaxis = new HashSet<Taxi>();
	}
	
	public synchronized void notifyAvailable(Taxi taxi) {
		availableTaxis.add(taxi);
	}
	/** 获取某个时刻，整个车队的完整快照 **/
	public synchronized Image getImage() {
		Image image = new Image();
		for (Taxi t : taxis) {
			image.drawMarker(t.getLocation());
		}
		return image;
	}
}
```

看看上面的代码，我们就知道setLocation和notifyAvailable方法都是同步方法，调用setLocation的线程首先获取Taxi上的锁，然后获取Dispatcher的锁。getImage方法先获取Dispatcher上的锁，再获取Taxi上的锁，这两个方法被不同的线程调用时容易产生死锁，相信现在的滴滴以及快的绝对不是这么干的。

下面我看看如何通过开放调用来避免这个死锁的问题。

```java
class Taxi {
	@GuardedBy("this")private Point location, destination;
	private final Dispatcher dispatcher;
	public Taxi(Dispatcher dispatcher) {
		this.dispatcher = dispatcher;
	}
	/** 获取出租车的位置 **/
	public synchronized Point getLocation() {
		return location;
	}
	/** 设置出租车的位置 **/
	public void setLocation(Point location) {
		boolean reachedDestination = false;
		/** 缩小缩的范围 把方法变成开发调用 **/
		synchronized(this) {
			this.location = location;
			reachedDestination = location.equals(distination);
		}
		
		if (reachedDestination) {
			dispatcher.notifyAvailable(this);
		}
	}
}
```

```java
class Dispatcher {
	@GuardedBy("this") private final Set<Taxi> taxis;
	@GuardedBy("this") private final Set<Taxi> availableTaxis;
	
	public Dispatcher() {
		taxis = new HashSet<Taxi>();
		availableTaxis = new HashSet<Taxi>();
	}
	
	public synchronized void notifyAvailable(Taxi taxi) {
		availableTaxis.add(taxi);
	}
	/** 获取每辆出租车不同时刻的位置 **/
	public Image getImage() {
		Set<Taxi> copy;
		synchronized(this) {
			copy = new HashSet<Taxi>(taxis);
		}
		Image image = new Image();
		for (Taxi t : taxis) {
			image.drawMarker(t.getLocation());
		}
		return image;
	}
}
```

通过上面的改造，我们把setLocation和getImage都变成了开放调用，与那些持有锁时调用外部方法的程序相比，更容易对依赖的开发调用的程序进行死锁分析。

在编写并发程序的时候，一定要注意一个思想，加锁范围最小化。
