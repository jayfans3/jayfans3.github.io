---
layout: post
title: Hbase物理架构
categories:
- 人
tags:
- 书
---



#Hbase物理架构

概念：

- hfile23
- compression
- split
- append,compaction


-hfile

- 随机访问block:1time datablock64k+metablock约100%命中:(2time bloomblock128k+1time indexblock128k)



