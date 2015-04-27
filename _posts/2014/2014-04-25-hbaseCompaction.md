---
layout: post
title: hbaseCompaction
categories:
- 人
tags:
- 书
---



#hbaseCompaction

上层触发：意义：合并文件，清除删除，过期，多余版本

1. Memstoreflush
2. rs.Compaction Checker
3. HBaseAdmin.flush
4. CompactTool

##Memstoreflush序列：

- **buffer->flush->merge的Log-Structured Merge-Tree**
- 1store-memstore1
- 进程：MemStoreFlusher->flushQueue：flush region请求
- 逻辑：->hbase.hstore.blockingStoreFiles>7?
  (hbase.hstore.blockingWaitTime>90？(flush region:
   region.split（split高与compaction）?:**CompactionSplitThread.compaction!**))
	->flushQuece

#####CompactSplitThread选择大小池excute
-----
- **1store->1CompactionRequest**:CompactSplitThread.compaction: 
- CompactionRequest:hfiles:large/small compaction ->两个池pool
- 池的选择阙值->**hbase.regionserver.thread.compaction.throttle**:2 * hbase.hstore.compaction.max * hbase.hregion.memstore.flush.size=2*10*128MB = 2.5GB
- compact:CompactionRunner.run(request)->1.gethfiles2.compaction

####底层存储引擎stripeStoreEngine
-----

######上下文初始化：HStore::requestCompaction->stripesotreEngine:CompactionContext

- 选择策略：compactionPolicy.selectCompaction()->hbase.hstore.defaultengine.compactionpolicy.class：ExploringCompactionPolicy.class
- 过滤出比compacting更年轻的hfile(依据：seq,filesize,bulktime,pathname):多个seq合并取最大

######small compation：ttl,部分，不处理已删除，多版本
- 合并ttl到期的hfile：minVersion=0||storefile.maxTimeStamp + store.ttl < now_timestamp
- 合并最大的hbase.hstore.compaction.max.size，超过过滤不合并
- 低于minFilesToCompact，忽略
- 不要bulKload hfile
- 枚举nhfile区间[hbase.hstore.compaction.min(默认3), hbase.hstore.compaction.max(默认10)]
- 总和小于hbase.hstore.compaction.max.size,超过最后的hfile去掉
- nhfile方差不能太大
- 减IO:删除最多的文件同时这些文件的大小总和小

######执行compaction:


######小升大
	 // Force a major compaction if this is a user-requested major compaction,
	 // or if we do not have too many files to compact and this was requested
	 // as a major compaction.
	 // Or, if there are any references among the candidates.



##Compaction Checker


- 服务级：compactionChecker:hbase.server.thread.wakefrequency=10000s=2:46：40
- 低优先级：HStore中StoreFIles的个数 – 正在执行Compacting的文件个数 > minFilesToCompact

####major触发 compaction:store所有一个hfile
- major_compact
- **间隔时间**：after (now - min(StoreFile.timestamp)) >"hbase.hregion.majorcompaction" + rand() *hbase.hregion.majorcompaction.jitter
- hbase.hregion.majorcompaction = 0可以关闭系统CompactionChecker.major 不能关闭用户的
- storefiles和filesToCompact文件个数相同：枚举数=hfile数 compactSelection.getFilesToCompact().size() < this.maxFilesToCompact
- 大于这maxCompactSize，过滤掉不合并
- smart算法：fileSizes[start] > Math.max(minCompactSize, (long)(sumSize[start+1]*r )，那么继续start++
- hbase.hstore.compaction.ratio 1.2F
- hbase.hstore.compaction.ratio.offpeak 5.0F 与下面两个参数联用 
- hbase.offpeak.start.hour -1 设置hbase offpeak开始时间[0,23]
- hbase.offpeak.end.hour -1 设置hbase offpeak结束时间 [0,23]




####序列：算文件

- 没有compacting的加入Candidates队列
- 算.compactSelection成CompactSelection对象
- 合并ttl到期的hfile：minVersion=0||storefile.maxTimeStamp + store.ttl < now_timestamp
- StoreScanner.ScanQueryMatcher.ColumnTracker（MAJOR_COMPACT,MINOR_COMPACT,USER_SCAN）
- 5：hbase.regionserver.thread.compaction.large，1hbase.regionserver.thread.compaction.large1，
- hbase.regionserver.thread.compaction.large




hbase.hregion.majorcompaction
hbase.hstore.compaction.max10
hbase.hstore.compactionThreshold3 ---5





影响：

1. store更新锁占用和snapshot的一样
2. flush整体的region级所有store对应的memorystroe的刷操作，compaction是store级操作粒度小
3. **flush为保持memorystore里连续的key区间，对应到hlog上一致性的key。**
4. **scan执行统一，hlog回放找flush_marker.replay**







- 合并最大的hbase.hstore.compaction.max.size
- 不要bulKload hfile
- 枚举nhfile区间[hbase.hstore.compaction.min(默认3), hbase.hstore.compaction.max(默认10)]
- 总和小于hbase.hstore.compaction.max.size,超过最后的hfile去掉
- nhfile方差不能太大
- 减IO:删除最多的文件同时这些文件的大小总和小