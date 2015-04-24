---
layout: post
title: Hbase进程架构
categories:
- 人
tags:
- 书
---



#Hbase进程架构

概念：

- master所有进程
- zk
- regionserver所有进程
- mvcc
- 各个级锁
- 归并排序
- 双向队列
- 路由策略
- blancer:robin,否定hash,cost-based
- 代价
- HA
- 