---
layout: post
title: slider中到底谁会去调用rpc??
categories:
- 逻辑与现象
- slider 服务设计
tags:
- slider
- 服务
- yarnrpc
---

slider中到底谁会去调用rpc??
=========

客户端最上抽象：


除了客户端基本操作以外,比如常规删除zk,hdfs,等等，到底谁会去调用rpc?


RPC的最基础实现是
使用rpc protocol的上层为：connet

使用connect的上层为：命令和其他抽象，freeze,flex,bondtoCluster

使用boundtoCluster的上层为：createClusterOperations

那么服务到底做了什么

![slideryarnrpc](/images/Image.png "slideryarnrpc客户端操作")