---
layout: post
title: sparkScheduler
categories:
- 逻辑与现象
- spark
tags:
- spark
- yarn
---


 sparkScheduler
============

spark运行方式：

- Local[N]
- Local cluster[worker,core.memory]
- sprk://localhost:prot
- mesos://host:port
- yarn standalone/cluster
- yarn client


资源调度：

- DAGScheduler（依赖，多个调度，shuffle来决定，数据本地性）TaskSet->TaskScheduler->starttask,monitor
- DAGScheduler所有模式一样，taskScheduler调度任务给计算资源不一样

##taskScheduler
###成对的TaskSchduler/SchedulerBackEnd与DAGschduler交互用

实现部分代码：

		private[spark] trait TaskScheduler {
				//	提交待运行任务及
				//去掉所有任务
				//DAGScheduler设置
				//并行度
				}
 		//继承
		private[spark] class TaskSchedulerImpl(
		val sc: SparkContext,
		val maxTaskFailures: Int,
		isLocal: Boolean = false)
		extends TaskScheduler with Logging
		def resourceOffers(offers: Seq[WorkerOffer]): Seq[Seq[TaskDescription]]
		def statusUpdate(tid: Long, state: TaskState, serializedData: ByteBuffer)
		
		private[spark] trait ExecutorBackend {
		def statusUpdate(taskId: Long, state: TaskState, data: ByteBuffer)
		}	
	




		//收到服务
		SchedulerBackEnd.reciveOffers()

		//运行服务
		Excutor->TaskRunner.run
		



		//运行池
		val threadPool = Utils.newDaemonCachedThreadPool("Executor task launch
		worker")
		private val runningTasks = new ConcurrentHashMap[Long, TaskRunner]
		def launchTask(context: ExecutorBackend, taskId: Long, serializedTask:
		ByteBuffer) {
		val tr = new TaskRunner(context, taskId, serializedTask)
		runningTasks.put(taskId, tr)
		threadPool.execute(tr)
		}
		


##模式：

##local:
###使用
	- ./bin/run-example org.apache.spark.examples.SparkPi local
	- ./bin/spark-submit \--class my.main.ClassName --master local[8] \ my-app.jar

###实现：

- TaskSchedulerImpl->LocalBackend.receiveOffers()->Excutor->TaskRunner.run

- **LocalActor AkkaActor**？？

##standalone:
###使用
	- ./sbin/start-master.sh
	- ./bin/spark-class org.apache.spark.deploy.worker.Worker spark:// MasterURL:PORT
	- conf/slave文件|spark-env.sh|sbin/*.sh
	- ./bin/run-example org.apache.spark.examples.SparkPi spark://10.0.2.31:7077
	-new: ./bin/spark-submit  --class my.main.ClassName --master spark://mycluster:7077 --executor-memory 20G  --total-executor-cores 100  my-app.jar

###实现

- TaskSchedulerImpl-**配合**>**SparkDeploySchedulerBackend** extends with **CoarseGrainedSchedulerBackend** ,with Akka Actor 控制excutor

序列：
#重要start SparkDeploySchedulerBackend->
#SparkDeploySchedulerBackend->client->master->worker->CoarseGrainedExecutorBackend-#>register SparkDeploySchedulerBackend->Driver Actor
#SparkDeploySchedulerBackend->客户端actor

##local cluster: 伪分布
###使用
	./bin/run-example org.apache.spark.examples.SparkPi local-cluster[2,2,1024]
###实现就是local standalone

##mesos 略

##yarn standalone0.9->yarn cluster1.0

###使用
-	 package：SPARK_HADOOP_VERSION=2.2.0 SPARK_YARN=true sbt/sbt assembly
-	 你的程序打包
-	 spark.yarn.submit.file.replication=10
-	 spark.yarn.preseve.stagin.files=ture
-	 spark.yarn.scheduler.heartbeat.interval-ms:5
-	 spark.yarn.max.worker.failures=?
- 使用：

		$ SPARK_JAR=./assembly/target/scala-2.10/spark-assembly-0.9.1-hadoop2.2.0.jar \
		./bin/spark-class org.apache.spark.deploy.yarn.Client \
		--jar examples/target/scala-2.10/spark-examples-assembly-0.9.1.jar \
		--class org.apache.spark.examples.SparkPi \
		--args yarn-standalone \
		--num-workers 3 \
		--master-memory 4g \
		--worker-memory 2g \
		--worker-cores 1

		new:

		HADOOP_CONF_DIR=XX /bin/spark-submit \
		--class my.main.ClassName \
		--master yarn-cluster \
		--executor-memory 20G \
		--num-executors 50 \
		my-app.jar

###实现：
- 提交app
- yarnrpc
- amipfiter
- driver线程启动application->init sparkcontext
- **start yarnallcator**
- yarnclusterschduler.postStartHook()->notify am inited sparkcontext ???
- yarnrpc.registertoAm
- allocator.numExcutorsContainer
- ExecutorRunnable->CoarseGrainedExecutorBackend->jobliststart
-  Monitor:akka->CoarseGrainedScheduler
- YARNClusterScheduler ，我真没见过这东西！，只是一个简单的TaskSchedulerImpl的实现加入等待excutorlogic,名字很挺狠。。。
- TaskSchedulerImpl->YARNClusterScheduler->**CoarseGrainedSchedulerBackend**

##yarn client 
###使用：不用client，直接配path!!  ’类am sparkcontext‘跑本地,Spark Shell 需要交互，调试类型，常驻本地
	 
	 SPARK_JAR
	 SPARK_WORKER_INSTANCES 2
	 SPARK_WORKER_CORES 1
	 SPARK_WORKER_MEMORY 1G
	 SPARK_MASTER_MEMORY 512 MB
	 SPARK_YARN_APP_NAME 
	 SPARK_YARN_QUEUE 'default'
	 SPARK_YARN_DIST_FILES
	 SPARK_YARN_DIST_ARCHIVES

	SPARK_JAR=./assembly/target/scala-2.10/spark-assembly-0.9.1-hadoop2.2
	.0.jar \
	SPARK_YARN_APP_JAR=examples/target/scala-2.10/spark-examples-assembly
	-0.9.1.jar \
	./bin/run-example org.apache.spark.examples.SparkPi yarn-client

###实现：
- am两件事：1.containe资源申请 2.维护driver调度
- 它把am的调度放在本地了，与executorbackend通信

序列：

	- initSparkContext
	  MapOutputTracker->MapOutputTracker->taskScheduler(选择scheduler(YarnClientClusterScheduler),backend(YarnClientSchedulerBackend))->dagScheduler
	- .runApp->ExecutorLauncher  
	   amClient->register->driver->MonitorActor--CoarseGrainedSchedulerBackend


maredue:vs 

- task 进程级 线程级


##使用scala提交任务

- scala写个
- ***.sbt

name := "Spark application in Scala" 
version := "1.0" 
scalaVersion := "2.10.4"
libraryDependencies += "org.apache.spark" %% "spark-core" % "1.0.0" 
resolvers += "Akka Repository" at "http://repo.akka.io/releases/" 

- sbt/sbt package
- bin/spark-submit --class "scala.Test"  --master local[4]
target/scala-2.10/simple-project_2.10-1.0.jar 



| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |