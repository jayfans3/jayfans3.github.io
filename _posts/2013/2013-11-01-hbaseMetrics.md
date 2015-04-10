---
layout: post
title: 关于hbaaes0.94 监控源码框架分享
categories:
- 监控
tags:
- hbase
- metrics
---



#关于hbaaes0.94 监控源码框架分享

	1. 0.95后引入metrics2 framework 采用source sink框架来统计数据,引入metric core( http://blog.csdn.net/scutshuxue/article/details/8351810)
	2. 此处讨论0.94的框架
	3. 0.94 依然可用

hbase0.95相关文献：

（api）This package provides a framework for metrics instrumentation and publication.

The framework provides a variety of ways to implement metrics instrumentation easily via the simple MetricsSource interface or the even simpler and more concise and declarative metrics annotations. The consumers of metrics just need to implement the simple MetricsSink interface. Producers register the metrics souces with a metrics system, while consumers register the sinks. A default metrics system is provided to marshal metrics from sources to sinks based on (per source/sink) configuration options. All the metrics are also published and queryable via the standard JMX MBean interface. This document targets the framework users. Framework developers could also consult the design document for architecture and implementation notes.


(git-0.95 changed)

0.95:

New Feature
HBASE-4050 Update HBase metrics framework to metrics2 framework
..

	[HBASE-6405] - Create Hadoop compatibilty modules and Metrics2 implementation of replication metrics
	[HBASE-6408] - Naming and documenting of the hadoop-metrics2.properties file
	[HBASE-6409] - Create histogram class for metrics 2
	[HBASE-6410] - Move RegionServer Metrics to metrics2
	[HBASE-6411] - Move Master Metrics to metrics 2
	[HBASE-6412] - Move external servers to metrics2 (thrift,thrift2,rest)
	[HBASE-6414] - Remove the WritableRpcEngine & associated Invocation classes
	[HBASE-6496] - Example ZK based scan policy
	[HBASE-6501] - Integrate with unit-testing tools of hadoop's metrics2 framework
	[HBASE-7262] - Move HBaseRPC metrics to metrics2

0.94的架构树

inteface：

	* org.apache.hadoop.metrics.MetricsContext
	* org.apache.hadoop.metrics.MetricsRecord


	* org.apache.hadoop.ipc.metrics.RpcMgtMBean


	* org.apache.hadoop.metrics.Updater

class:

	* org.apache.hadoop.metrics.spi.AbstractMetricsContext (implements org.apache.hadoop.metrics.MetricsContext)
	* 
		* org.apache.hadoop.metrics.spi.CompositeContext
		* org.apache.hadoop.metrics.file.FileContext
		* org.apache.hadoop.metrics.ganglia.GangliaContext
		* org.apache.hadoop.metrics.spi.NullContext
		* org.apache.hadoop.metrics.spi.NullContextWithUpdateThread




	* org.apache.hadoop.metrics.jvm.EventCounter
	* 



	* org.apache.hadoop.metrics.ContextFactory



	* org.apache.hadoop.metrics.util.MBeanUtil
	* org.apache.hadoop.metrics.util.MetricsBase
		* org.apache.hadoop.metrics.util.MetricsIntValue
		* org.apache.hadoop.metrics.util.MetricsLongValue
		* org.apache.hadoop.metrics.util.MetricsTimeVaryingInt
		* org.apache.hadoop.metrics.util.MetricsTimeVaryingLong
		* org.apache.hadoop.metrics.util.MetricsTimeVaryingRate
		* 


	* org.apache.hadoop.metrics.util.MetricsDynamicMBeanBase (implements javax.management.DynamicMBean)

		* org.apache.hadoop.ipc.metrics.RpcActivityMBean

	* org.apache.hadoop.metrics.spi.MetricsRecordImpl (implements org.apache.hadoop.metrics.MetricsRecord)
	* 

	* org.apache.hadoop.metrics.util.MetricsRegistry
	* org.apache.hadoop.metrics.MetricsUtil
	* org.apache.hadoop.metrics.spi.MetricValue
	* 



	* org.apache.hadoop.metrics.spi.OutputRecord
	* 



	* org.apache.hadoop.metrics.jvm.JvmMetrics (implements org.apache.hadoop.metrics.Updater)
	* 



	* org.apache.hadoop.ipc.metrics.RpcMetrics (implements org.apache.hadoop.metrics.Updater)
	* 



	* org.apache.hadoop.metrics.MetricsException

![jiagou](/images/3/IMG_20131127_101723.jpg)


hbase框架：

package:
org.apache.hadoop.hbase.client.metrics,
org.apache.hadoop.hbase.metrics
 org.apache.hadoop.hbase.master.metrics
org.apache.hadoop.hbase.metrics.file,
 org.apache.hadoop.hbase.metrics.histogram
org.apache.hadoop.hbase.regionserver.metrics
 org.apache.hadoop.hbase.rest.metrics
inteface：
org.apache.hadoop.hbase.regionserver.metrics.SchemaMetrics.SchemaAware
org.apache.hadoop.hbase.io.hfile.HFile.Reader
     org.apache.hadoop.hbase.io.hfile.HFile.CachingBlockReader
class:

	* org.apache.hadoop.metrics.spi.AbstractMetricsContext (implements org.apache.hadoop.metrics.MetricsContext)
	* 
		* org.apache.hadoop.metrics.file.FileContext
		* 
			* org.apache.hadoop.hbase.metrics.file.TimeStampingFileContext





	* org.apache.hadoop.hbase.thrift.HbaseHandlerMetricsProxy (implements java.lang.reflect.InvocationHandler)
	* org.apache.hadoop.hbase.metrics.HBaseInfo




	* org.apache.hadoop.hbase.ipc.HBaseRPC


	* org.apache.hadoop.hbase.ipc.HBaseRpcMetrics (implements org.apache.hadoop.metrics.Updater)


	* org.apache.hadoop.hbase.regionserver.wal.HLog.Metric


	* org.apache.hadoop.hbase.master.metrics.MasterMetrics (implements org.apache.hadoop.metrics.Updater)



	* org.apache.hadoop.metrics.util.MetricsBase

		* org.apache.hadoop.hbase.metrics.ExactCounterMetric
		* org.apache.hadoop.hbase.metrics.histogram.MetricsHistogram
		* org.apache.hadoop.hbase.metrics.MetricsRate
		* org.apache.hadoop.hbase.metrics.MetricsString
		* org.apache.hadoop.metrics.util.MetricsTimeVaryingRate

			* org.apache.hadoop.hbase.metrics.PersistentMetricsTimeVaryingRate


	* org.apache.hadoop.metrics.util.MetricsDynamicMBeanBase (implements javax.management.DynamicMBean)



	* org.apache.hadoop.hbase.ipc.HBaseRPCStatistics
	* org.apache.hadoop.hbase.metrics.MetricsMBeanBase
		* org.apache.hadoop.hbase.metrics.HBaseInfo.HBaseInfoMBean
		* org.apache.hadoop.hbase.master.metrics.MasterStatistics
		* org.apache.hadoop.hbase.regionserver.metrics.RegionServerStatistics
		* org.apache.hadoop.hbase.replication.regionserver.ReplicationStatistics
		* org.apache.hadoop.hbase.rest.metrics.RESTStatistics

	* org.apache.hadoop.hbase.regionserver.metrics.RegionServerDynamicStatistics



	* org.apache.hadoop.hbase.regionserver.metrics.OperationMetrics



	* org.apache.hadoop.hbase.regionserver.metrics.RegionMetricsStorage
	* org.apache.hadoop.hbase.regionserver.metrics.RegionServerDynamicMetrics (implements org.apache.hadoop.metrics.Updater)
	* org.apache.hadoop.hbase.regionserver.metrics.RegionServerMetrics (implements org.apache.hadoop.metrics.Updater)


	* org.apache.hadoop.hbase.replication.regionserver.ReplicationSourceMetrics (implements org.apache.hadoop.metrics.Updater)


	* org.apache.hadoop.hbase.replication.regionserver.ReplicationSinkMetrics (implements org.apache.hadoop.metrics.Updater)


	* org.apache.hadoop.hbase.replication.regionserver.ReplicationSourceMetrics (implements org.apache.hadoop.metrics.Updater)


	* org.apache.hadoop.hbase.rest.metrics.RESTMetrics (implements org.apache.hadoop.metrics.Updater)


	* org.apache.hadoop.hbase.client.metrics.ScanMetrics (implements org.apache.hadoop.io.Writable)

      org.apache.hadoop.hbase.regionserver.metrics.SchemaConfigured (implements org.apache.hadoop.hbase.io.HeapSize, org.apache.hadoop.hbase.regionserver.metrics.SchemaMetrics.SchemaAware)


	* org.apache.hadoop.hbase.regionserver.metrics.SchemaMetrics


	* org.apache.hadoop.hbase.thrift.ThriftMetrics (implements org.apache.hadoop.metrics.Updater)


MetricsContext

通过ContextFactory我们可以获得一个MetricsContext对象，它保存这一组metrics的上下文信息，每个context都可以启动一个monitor线程来按一定周期来收集和发送这些数据。前面我们也提到了hadoop可以提供写文件，发送ganglia等方式来发送metrics，就是通过继承

AbstractMetricsContext产生不同的Context的类来实现的。具体要采用那种方式来发送数据可以通过配置文件来确定，确定使用哪种context

Updater

Updater是数据收集的主体，这个接口最重要的是doUpdates方法。将应用实现的updater类注册到MetricsContext中，context的monitor线程就会定期调用updater的doUpdates方法来抓取数据。通常在doUpdates里我们会对系统的各种metrics做初步计算处理并push到MetricsRecord中。

MetricsRecord

顾名思义它是一个时间周期下的一条数据，每个context下可以包含多个MetricsRecord。Context调用doUpdates收集到MetricsRecord后，再将它发送给文件或者ganglia。对于ganglia来说每一个监控点名称格式就是：context名称. Record名称.metrics名称。

MetricsBase

	所有具体的metrics数据类都会继承MetricsBase，常用的有以下几种：
	MetricsIntValue：整型数值
	MetricsLongValue：长整型数值
	MetricsTimeVaryingInt：一个时间周期内整型累积值，push到MetricsRecord后从0开始累积。
	MetricsTimeVaryingLong：一个时间周期内长整型累积值
	MetricsTimeVaryingRate：保存了操作所花费的时间和该时间内的操作次数。最终发送出去会是两个数值：一个时间周期内的总操作次数和每个操作所花费的平均时间（操作总时间/操作次数）。该metrics内部还保存了单次操作时间的min和max。
	Hbase也扩展了几种metrics类型：
	MetricsAvgValue：记录用户设置的值及设置的次数，最终输出的是平均值，收集完清零。
	MetricsIntervalInt：一个时间周期内整型数值，收集完清零。
	MetricsRate：一个时间周期内的数据/时间周期的长度，收集完清零。
	MetricsString：一个字符串，主要用于JMX，ganglia不支持。
