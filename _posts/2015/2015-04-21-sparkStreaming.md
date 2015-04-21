---
layout: post
title: spark流
categories:
- 逻辑与现象
- spark
tags:
- spark
- sql
- sparksql
---

spark流
============

#概念

- DsStream rdd@updatetime
- task:receiver-input DStream-core/slot（一个container包含的线程业务个数，比如计算和监控，流处理 一共三个）
- checkpoint容错
- split interval 切分周期--切后算成rdd
- cache:persist vs 内存+序列化
- 空rdd:

		socketStream.foreachRDD(rdd => { 
		  if(!rdd.partitions.isEmpty){ 
		 rdd.saveAsTextFile(outputDir)	
		 } 	
		}) 
- updateStateByKey:StateDStream: 每次的增量rdd进行一次cogroup->update function,合并有序，增量计算可以并发
- 窗口决定的参数：batch size+window length+sliding interval。batchs rdd.cogroup UnionRDD
-**cogroup**: RDD[K, V]和RDD[K, U]合成RDD[K, List[V], List[U]]，List[U]一般size是1，理解为oldvalue，即RDD[K, batchValueList, Option[oldValue]]。

- 

     /**
	   * Apply a function to each RDD in this DStream. This is an output operator, so
	   * 'this' DStream will be registered as an output stream and therefore materialized.
	   */
	  def foreachRDD(foreachFunc: (RDD[T], Time) => Unit) {
	    new ForEachDStream(this, context.sparkContext.clean(foreachFunc, false)).register()
	  }

[https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/SqlNetworkWordCount.scala](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/SqlNetworkWordCount.scala)

- cp:intervar设定，batchs一个cp,同步，没有state操作不用cp


#sequences

- instance:streamingContext->sparkContext->spark master url->spark engine->excutor
- ReceiverInputDsStream：var lines=ssc.socketTextStreaming("localhost",9999)
- task上得到Receiver:var receiver=lines.getReceiver()

  		spark.streaming.receiver.maxRate

- 实现：receiver.onStart{},receiver.onStop(){} 存资源和释放资源实现
- trait receiverSupervisor with Receiver .store(ByteBuffer,Iterator)监控停止启动重启error （receiverSupervisor:BlockManager.storageLevel）
- ->存sequence：receiverSupervisor.putBlock->BlockManager存数据->ReceiverTracker.AddBlock消息
- ->收到消息ReceiverTracker->ReceivedBlockTracker维护接收的BlockInfo
- ->取sequence:ReceiverTracker->streamId

#scheduler

- spark.streaming.concurrentJobs =1 
- JobScheduler:Context中初始化->jobscheduler.start->
- ->receiverTracker.start(workder.receiver 监听数据,block地址),jobgenerator.start（生成数据和时间来生成任务描述）
- JobGenerator：JobSet s：ReceiverTracker.allocateBlocksToBatch->DStream.generateJob->DStream.getOrCompute(time)->if time rdd 没有，批次时间间隔整数倍->ds.compute->rdd.persist->checkpoint->JobScheduler.submitJobSet->**JobGenerator.DoCheckpoint元数据**->JobCompleted（job）->handleJobCompletion(job)->onBatchCompletion(time)->clearMetadata


#datasource

- file, socket, akka, RDDs
- Twitter, Kafka, Flume


#容错

- work失败:外部data低延迟输入到其他目录/网络获取失败重入
- drvier失败：元数据做checkpoint/data lost/wal
- data lost：wal 降低吞吐量

#调优

- batchsize:log4j和listener实时监控速度
- spark.cleaner.ttl: metadta
- spark.streaming.unpersist:rdd
- cms
- fullgcbuforecompaction

#point





kafka:

- 一个话题（topic）可以有N个分区
- 