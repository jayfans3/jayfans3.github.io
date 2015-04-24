---
layout: post
title: Hbase内存架构
categories:
- 人
tags:
- 书
---



#Hbase内存架构

概念：

- memstore:0.2
- lsm b+变种
- mslab +LTAB 碎片
- blockcache:0.6 lru backetcache加载时机一致性
- hlog wal: write的容错replay，>1 WAL
- cms,fullgc 优化
- 


