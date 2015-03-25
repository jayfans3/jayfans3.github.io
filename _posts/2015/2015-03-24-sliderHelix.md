---
layout: post
title: slider注册服务的外部视图展示相关的设计
categories:
- 逻辑与现象
- slider 服务设计
tags:
- slider
- 服务
- helix
---


lift extenal view
===========


####helix:

- add cluster
- addnode
	helix-admin.sh --zkSvr localhost:2199  --addNode MYCLUSTER localhost:12915
- Define the Resource and Partitioning
- 
	hbase 6个版本的运行
	helix-admin.sh --zkSvr localhost:2199 --addResource MYCLUSTER hbase 6 MasterSlave

	spark 10个分区的运行
	helix-admin.sh --zkSvr localhost:2199 --addResource MYCLUSTER spark 6 MasterSlave

/user/ocean/.slider/cluster/s6_1/database/data/default/A
zk定义完成
![a](/images/0/4.png)
- 管理启动
	run-helix-controller.sh --zkSvr localhost:2199 --cluster MYCLUSTER 2>&1 > /tmp/controller.log &

- 启动节点

	start-helix-participant.sh --zkSvr localhost:2199 --cluster MYCLUSTER --host localhost --port 12913 --stateModelType MasterSlave 2>&1 > /tmp/participant_12913.log

- curd

http://helix.apache.org/0.7.1-docs/Quickstart.html

参考体验来添加行为 服务发现和文件实时同步等

**pfs:文件实时同步 删除添加频繁但是更新并发需求不大。视频、图片、邮件附件大量小文件 hdfs不适用**



	* Large number of files but each file is relatively small
	* Access is limited to create, delete and get entire files
	* No updates to files that are already created

http://helix.apache.org/0.7.1-docs/recipes/rsync_replicated_file_store.html


	* CRD access to large number of small files
	* Scalability: Files should be distributed across a large number of commodity servers based on the storage requirement
	* Fault-tolerance: Each file should be replicated on multiple servers so that individual server failures do not reduce availability
	* Elasticity: It should be possible to add capacity to the cluster easily

