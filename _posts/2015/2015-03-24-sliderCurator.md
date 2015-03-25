---
layout: post
title: slider curator
categories:
- 逻辑与现象
- slider 使用
tags:
- slider
- 服务
- 使用
---


Curator uses Fluent Style

Curator was open-sourced by Netflix on GitHub as an Apache 2.0 licensed project in July 2011. During this time Curator has been formally released many times and has gained widespread adoption.
The Curator Incubator Proposal was presented in February 2013 and was formally accepted into the Incubator in March 2013.
Curator graduated to a Top Level Project in September, 2013.




client.create().forPath("/my/path", myData)

RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3)
CuratorFramework client = CuratorFrameworkFactory.newClient(zookeeperConnectionString, retryPolicy);
client.start();

example---》下面： 

Service Discovery

	* Services to register their availability
	* Locating a single instance of a particular service
	* Notifying when the instances of a service change

