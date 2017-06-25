title: 如何在本地通过eclipse来调试HBase的源代码
date: 2014-06-07 22:36:16
tags: HBase
categories: 大数据
---

####一.概述
要想把HBase的源代码梳理清楚，需要在本地调试一下，可能直接看代码也能搞清楚，但是调试能让我们尽快搞清楚HBase的源代码，下面就来看一下如何在eclipse中调试HBase的源代码。

<!-- more -->

####二.编译HBase的源代码

1.HBase源代码的获取
你可以直接从HBase的官方网站上下载HBase的包，[HBase官方地址](http://hbase.apache.org/)
你也可以使用svn把hbase的代码签出
```
svn co http://svn.apache.org/repos/asf/hbase/tags/0.94.2/ hbase_sources
```

2.HBase源代码的编译
进入到HBase代码的目录下面，执行下面的命令
```
mvn clean install mvn clean install -DskipTests
mvn eclipse clean eclipse:eclipse
```
执行完上面的命令后，就把HBase的源码导入到eclipse中去。在执行maven命令的时候注意使用maven包自带的settings.xml，不要使用其他settings.xml文件，会导致编译不过的。

3.在eclipse中做相关的设置

在eclipse的Run Configurations中设置启动类
![设置启动类HMaster](http://bolinyoung.qiniudn.com/hbase-start.png)

在eclipse的Run Configurations中设置启动参数
![设置启动参数](http://bolinyoung.qiniudn.com/hbase-start-args.png)

在eclipse的Run Configurations中把conf文件放到classpath中
![把conf文件放到classpath中](http://bolinyoung.qiniudn.com/hbase-start-conf.png)

4.点击Run运行
出现了下面的错误
![xml版本校验出错](http://bolinyoung.qiniudn.com/hbase-xml-version.png)
观察出错日志，显示hbase-default.xml版本校验出错，搞不定，搜一把，有人说在hbase-site.xml增加下面的配置项
```xml
<property>
	<name>hbase.defaults.for.version.skip</name>
	<value>true</value>
	<description>
		Set to true to skip the 'hbase.defaults.for.version' check.
		Setting this to true can be useful in contexts other than
		the other side of a maven generation; i.e. running in an
		ide. You'll want to set this boolean to true to avoid
		seeing the RuntimException complaint: "hbase-default.xml file
		seems to be for and old version of HBase (@@@VERSION@@@), this
		version is X.X.X-SNAPSHOT"
	</description>
</property>
```
索性试一下，此错误消失了，又来一错误
![HBase ZK ERROR](http://bolinyoung.qiniudn.com/hbase-zk-error.png)
这个错之前遇到过是由于zookeeper没启动导致，一直以为运行HMaster类，会自己启动zookeeper，所以一直在网上找答案，结果找到的答案都是错的，于是怀疑zookeeper在自己的电脑上压根就没有启动起来,所以需要想办法在本地启动zookeeper，我们看到hbase源代码中有一个HQuorumPeer类，这个类就是hbase-zookeeper的启动类，该类中有一个main方法，我们在eclipse中以application的方式启动。
![hbase zk启动类](http://bolinyoung.qiniudn.com/hbase-zk-start.png)
把conf下面的配置文件设置到classpath中
![设置conf到classpath](http://bolinyoung.qiniudn.com/hbase-zk-classpath.png)
这样我们就可以在eclipse中启动zookeeper啦。接下来我们再次启动HMaster
![启动HMaster控制台不断输出错误日志](http://bolinyoung.qiniudn.com/hbase-region-server-error.png)
又出错了，看看错误日志，我们发现是RegionServer没有启动好，使用jps可以发现没有RegionServer的进程，虽然日志的级别是INFO的，回头一看确实没有启动过RegionServer，所以启动一下RegionServer啦，我们还是在eclipse中以application的方式去启动RegionServer
![HBase RegionServer的启动配置](http://bolinyoung.qiniudn.com/hbase-region-server.png)
同样需要把conf文件加入到eclipse的classpath中，方法和前面一样，这里就不用再重复了。配置好就启动,然后发现HMaster的控制台INFO级别的日志停掉了。关于这个点我们稍微再深入一些，我们刚才的场景是启动了HMaster但是没有启动RegionServer，此时HMaster的控制台一直有INFO级别的日志输出，我们看一下ServerManager这个类中输出这段日志的代码
```java
/**
   * Wait for the region servers to report in.
   * We will wait until one of this condition is met:
   *  - the master is stopped
   *  - the 'hbase.master.wait.on.regionservers.maxtostart' number of
   *    region servers is reached
   *  - the 'hbase.master.wait.on.regionservers.mintostart' is reached AND
   *   there have been no new region server in for
   *      'hbase.master.wait.on.regionservers.interval' time AND
   *   the 'hbase.master.wait.on.regionservers.timeout' is reached
   *
   * @throws InterruptedException
   */
  public void waitForRegionServers(MonitoredTask status)
  throws InterruptedException {
    final long interval = this.master.getConfiguration().
      getLong(WAIT_ON_REGIONSERVERS_INTERVAL, 1500);
    final long timeout = this.master.getConfiguration().
      getLong(WAIT_ON_REGIONSERVERS_TIMEOUT, 4500);
    int minToStart = this.master.getConfiguration().
      getInt(WAIT_ON_REGIONSERVERS_MINTOSTART, 1);
    if (minToStart < 1) {
      LOG.warn(String.format(
        "The value of '%s' (%d) can not be less than 1, ignoring.",
        WAIT_ON_REGIONSERVERS_MINTOSTART, minToStart));
      minToStart = 1;
    }
    int maxToStart = this.master.getConfiguration().
      getInt(WAIT_ON_REGIONSERVERS_MAXTOSTART, Integer.MAX_VALUE);
    if (maxToStart < minToStart) {
        LOG.warn(String.format(
            "The value of '%s' (%d) is set less than '%s' (%d), ignoring.",
            WAIT_ON_REGIONSERVERS_MAXTOSTART, maxToStart,
            WAIT_ON_REGIONSERVERS_MINTOSTART, minToStart));
        maxToStart = Integer.MAX_VALUE;
    }

    long now =  System.currentTimeMillis();
    final long startTime = now;
    long slept = 0;
    long lastLogTime = 0;
    long lastCountChange = startTime;
    int count = countOfRegionServers();
    int oldCount = 0;
    while (
      !this.master.isStopped() &&
        count < maxToStart &&
        (lastCountChange+interval > now || timeout > slept || count < minToStart)
      ){

      // Log some info at every interval time or if there is a change
      if (oldCount != count || lastLogTime+interval < now){
        lastLogTime = now;
        String msg =
          "Waiting for region servers count to settle; currently"+
            " checked in " + count + ", slept for " + slept + " ms," +
            " expecting minimum of " + minToStart + ", maximum of "+ maxToStart+
            ", timeout of "+timeout+" ms, interval of "+interval+" ms.";
        LOG.info(msg);
        status.setStatus(msg);
      }

      // We sleep for some time
      final long sleepTime = 50;
      Thread.sleep(sleepTime);
      now =  System.currentTimeMillis();
      slept = now - startTime;

      oldCount = count;
      count = countOfRegionServers();
      if (count != oldCount) {
        lastCountChange = now;
      }
    }

    LOG.info("Finished waiting for region servers count to settle;" +
      " checked in " + count + ", slept for " + slept + " ms," +
      " expecting minimum of " + minToStart + ", maximum of "+ maxToStart+","+
      " master is "+ (this.master.isStopped() ? "stopped.": "running.")
    );
  }
```
重点关注这段代码中的while循环，这段代码不断轮寻RegionServer数目，只有启动起来的RegionServer数目>=hbase.master.wait.on.regionservers.mintostart这个while循环才能退出，不然while循环里面就会一直输出INFO的级别的日志，说没有RegionServer启动好，其实hbase.master.wait.on.regionservers.mintostart的默认值就是1.

至此，Zookeeper，HMaster，HRegionServer都已经启动好了，这里所说的HRegionServer和上面所说的RegionServer指的是同一个东西。接下来我们访问http://localhost:60010/master-status 就能看到相关的信息。

这样我们就可以在本地eclipse中debug一下HBase的源代码啦^=^。

####三.最后总结
我们发现在本地部署HBase，我们需要启动三个东西，Zookeeper,HMaster以及RegionServer。

zookeeper通过HMaster负责协调整个HBase集群，同时zookeeper保存了hbase中-ROOT-表的地址和HMaster的地址。HRegionServer也会把自己以Ephemeral方式注册到Zookeeper中，使得HMaster可以随时感知到各个HRegionServer的健康状态。

HMaster没有单点问题，HBase中可以启动多个HMaster，通过Zookeeper的Master Election机制保证总有一个Master运行，HMaster在功能上主要负责Table和Region的管理工作：
1. 管理用户对Table的增、删、改、查操作
2. 管理HRegionServer的负载均衡，调整Region分布
3. 在Region Split后，负责新Region的分配
4. 在HRegionServer停机后，负责失效HRegionServer 上的Regions迁移

HRegionServer主要负责响应用户I/O请求，向HDFS文件系统中读写数据，是HBase中最核心的模块。
HRegionServer内部管理了一系列HRegion对象，每个HRegion对应了Table中的一个Region，HRegion中由多个HStore组成。每个HStore对应了Table中的一个Column Family的存储，可以看出每个Column Family其实就是一个集中的存储单元，因此最好将具备共同IO特性的column放在一个Column Family中，这样最高效。