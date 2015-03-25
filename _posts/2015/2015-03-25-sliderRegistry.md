---
layout: post
title: slider注册服务
categories:
- 逻辑与现象
- slider 使用
tags:
- slider
- 服务
- 使用
---


#注册服务


看这个里面有pdf：

[umbrella yarn-913](https://issues.apache.org/jira/browse/YARN-913)

概念：
========

- 服务 an app server for hbase
- 服务类 注册内部的叫法
- 组件 服务的元素
- 服务实例 app实例
- 组件实例（regionserver ,master)
- endpoints 接入服务的接口比如hbase 的rest thrift等等 ipc port ipcinfoport等等
- 服务record 实例注册描述
- rm
- yarn app
- am
- container

绑定问题
--------
- ip 或host静态方式绑定组件实例行不通
- 
1. yarn实例id变化
2. 不能注册rest/zkpath/endpoints
3. 实例存活


服务
====
1. A client application looks up a dynamically deployed service instance whose user,
service class and instance name is known, e.g.

	"/users/joe/services/hbase/demo1", and retrieves the information needed to
connect to the service
2. A client application looks up a statically deployed Hadoop service

	Example: "/services/hdfs".
3. An Application Master enumerates all registered component instances, discovers their listed JMX ports, and, inits own web UI, offers links to these endpoints.
4. A user connects to a private HBase service instance at

	"/users/joe/services/hbase/demo1"
5. A user connects to the cluster’s HBase service at "/services/hbase".
6. A user looks up the binding information to a remote Hadoop cluster's filesystem at "/net/cluster4/services/hdfs". The registration information includes the webhdfs:// URL for the remote filesystem
7. A user lists their HBase service instances:

	ls /users/joe/services/hbase
8. User finds all Hbase services in the cluster:

	find -endpointField.api=org.apache.hbase
9. Possibly in future: looking up a service via DNS.

注册意义发现
----------------
1. 没有yarn
2. 注册信息要比实例存活长，解耦实例。
3. 组件发布info
4. 绑定实例
5. 组件实例使用注册
6. 不自动注册

注册关键点
============
1. 动态注册 （绑定并发现、绑定不随服务移动而移动或是因为HAfailover)
2. endpoint多样
3. 注册服务属性：ha,scale,Ubiquity
4. 分层命名
5. 跨语言
6. 读写控制
7. 远程读写


- Service Record: application. info
- persistence attribute policies 0-5
- Endpoint
- Address Type(path url )


eg:

	/users/devteam/tomcat/components/container-1408631738011-0001-01-000001 (ephemeral) 

	{
	"registrationTime" : 1408638082445,
	"id" : "container_1408631738011_0001_01_000001",
	"persistence" : "1",
	"description" : null,
	"external" : [ {
	"api" : "www",
	"addressType" : "uri",
	"protocolType" : "REST",
	"addresses" : [ [ "http://rack4server3:43572" ] ]
	} ],
	"internal" : [ {
	"api" : "jmx",
	"addressType" : "host/port",
	"protocolType" : "JMX",
	"addresses" : [ [ "rack4server3", "43573" ] ]
	} ]
	}


存取设计：

RegistryOperations
