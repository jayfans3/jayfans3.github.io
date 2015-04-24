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


##hfile

- 随机访问block:1time datablock64k+metablock约100%命中:(2time bloomblock128k+1time indexblock128k)
- KeyValue类中自带的字段占用大小约为50~60 bytes
- hfile:scanedblock,nonscanedblock,loadonopen,trailer
- rootindex二分查找/leaf/intermediate index chunk 二分查找->read hdfsblock->遍历key
- 

