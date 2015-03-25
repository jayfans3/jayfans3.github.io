---
layout: post
title: zookeeper注册服务可用性和性能问题
categories:
- 逻辑与现象
- CAP
tags:
- slider
- 服务
- ZK
- service registry
---


#zookeeper注册服务可用性和性能问题

##其实推销一个产品：eureka，真的不错。

zookeeper功能：
1.一致性分布式事务:锁，同步队列，同步
3.选举
4.服务发现


[https://issues.apache.org/jira/browse/ZOOKEEPER-1159](https://issues.apache.org/jira/browse/ZOOKEEPER-1159)
[https://issues.apache.org/jira/browse/ZOOKEEPER-1576](https://issues.apache.org/jira/browse/ZOOKEEPER-1576)

- 如果作为协调服务没问题，因为同步和数据调度是信息的一致性问题。
- 如果作为服务发现的功能，比如节点丢失或遇到网络分区问题就失去协调服务能力，但是服务仍然存在,用缓存弥补，破坏一致性。
- 服务发现初设计可用性，恢复力。
这个[https://github.com/Netflix/eureka/](https://github.com/Netflix/eureka/)


概念
----------
- 无选举
- 它可以在零停机的情况下处理更广泛的网络分区问题
- 服务心跳的概念：自我保护模式
- 服务器cache解决可用性
- 服务心跳、服务健康检查、自动发布及缓存刷新
- REST

