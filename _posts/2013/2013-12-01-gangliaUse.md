---
layout: post
title: hbaaes配置ganglia
categories:
- 监控
tags:
- hbase
- metrics
---



#hbase配置ganglia


指标：
manor->major compact.监控
split:hfile太大 不用
put->wal size-> too many logs->regionserver flush.regionserver 停用
blocksize.uplimit.
blancer 只要regionserver。 hash均匀。
regionserver flush.


 ganglia使用hbase指标配置：
   
	hbase.sink.ganglia.metric.filter.exclude=^(.*table.*)|(\\w+metric)|(\\w+region\\w+)|(Balancer\\w+)|(\\w+Assign\\w+)|(\\w+percentile)|(\\w+max)|(\\w+median)|(\\w+min)|(MetaHlog\\w+)|(\\w+WAL\\w+)$
	hbase.sink.ganglia.period=10  
	*.source.filter.class=org.apache.hadoop.metrics2.filter.RegexFilter
	#*.source.filter.class=org.apache.hadoop.metrics2.filter.GlobFilter
	*.record.filter.class=${*.source.filter.class}
	*.metric.filter.class=${*.source.filter.class}
	*.period=10
	
	*.sink.ganglia.class=org.apache.hadoop.metrics2.sink.ganglia.GangliaSink31  
	hbase.sink.ganglia.servers=ip：port
这是配置ganglia输出指标的配置 配上重启就能往组播上吐数据
关于过滤数据：
	http://hadoop.apache.org/docs/current/api/org/apache/hadoop/metrics2/package-summary.html#filtering

大量指标过滤复杂支持regex patten。

hadoop source:

public class RegexFilter extends AbstractPatternFilter {

  @Override
  protected Pattern compile(String s) {
    return Pattern.compile(s);
  }
}

hbase官网参考指标：

	120.5. Most Important Master Metrics
	Note: Counts are usually over the last metrics reporting interval.
	hbase.master.numRegionServersNumber of live regionservers
	hbase.master.numDeadRegionServersNumber of dead regionservers
	hbase.master.ritCountThe number of regions in transition
	hbase.master.ritCountOverThresholdThe number of regions that have been in transition longer than a threshold time (default: 60 seconds)
	hbase.master.ritOldestAgeThe age of the longest region in transition, in milliseconds
	120.6. Most Important RegionServer Metrics
	Note: Counts are usually over the last metrics reporting interval.
	hbase.regionserver.regionCountThe number of regions hosted by the regionserver
	hbase.regionserver.storeFileCountThe number of store files on disk currently managed by the regionserver
	hbase.regionserver.storeFileSizeAggregate size of the store files on disk
	hbase.regionserver.hlogFileCountThe number of write ahead logs not yet archived
	hbase.regionserver.totalRequestCountThe total number of requests received
	hbase.regionserver.readRequestCountThe number of read requests received
	hbase.regionserver.writeRequestCountThe number of write requests received
	hbase.regionserver.numOpenConnectionsThe number of open connections at the RPC layer
	hbase.regionserver.numActiveHandlerThe number of RPC handlers actively servicing requests
	hbase.regionserver.numCallsInGeneralQueueThe number of currently enqueued user requests
	hbase.regionserver.numCallsInReplicationQueueThe number of currently enqueued operations received from replication
	hbase.regionserver.numCallsInPriorityQueueThe number of currently enqueued priority (internal housekeeping) requests
	hbase.regionserver.flushQueueLengthCurrent depth of the memstore flush queue. If increasing, we are falling behind with clearing memstores out to HDFS.
	hbase.regionserver.updatesBlockedTimeNumber of milliseconds updates have been blocked so the memstore can be flushed
	hbase.regionserver.compactionQueueLengthCurrent depth of the compaction request queue. If increasing, we are falling behind with storefile compaction.
	hbase.regionserver.blockCacheHitCountThe number of block cache hits
	hbase.regionserver.blockCacheMissCountThe number of block cache misses
	hbase.regionserver.blockCacheExpressHitPercentThe percent of the time that requests with the cache turned on hit the cache
	hbase.regionserver.percentFilesLocalPercent of store file data that can be read from the local DataNode, 0-100
	hbase.regionserver.<op>_<measure>Operation latencies, where <op> is one of Append, Delete, Mutate, Get, Replay, Increment; and where <measure> is one of min, max, mean, median, 75th_percentile, 95th_percentile, 99th_percentile
	hbase.regionserver.slow<op>CountThe number of operations we thought were slow, where <op> is one of the list above
	hbase.regionserver.GcTimeMillisTime spent in garbage collection, in milliseconds
	hbase.regionserver.GcTimeMillisParNewTime spent in garbage collection of the young generation, in milliseconds
	hbase.regionserver.GcTimeMillisConcurrentMarkSweepTime spent in garbage collection of the old generation, in milliseconds
	hbase.regionserver.authenticationSuccessesNumber of client connections where authentication succeeded
	hbase.regionserver.authenticationFailuresNumber of client connection authentication failures
	hbase.regionserver.mutationsWithoutWALCount
	Count of writes submitted with a flag indicating they should bypass the write ahead log
	
