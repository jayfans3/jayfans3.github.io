---
layout: post
title: slider 概念名词
categories:
- 现象与使用
- slider 服务设计
tags:
- slider
- 服务
---


slider 概念名词
--------------

[TOC]

------------------

###**优先概念**：
> 优先概念：客户端 服务端 提供端 注册端
> 优先概念: slider客户端编程，slider集群操作，yarn客户端编程，yarn编程操作
> 服务库
> 提供者就是配置或组装的，但不是管发射的
> 提供者服务
> 服务概念不一定非用事件库的东西，
> 序列化 框架 rpcMessage通信
> 火箭launched和发射器火箭台launch launcher抽象
> duration 持续时间
> ordinal service state
> 程序关闭钩子
> key概念 是set的钥匙 作为门栓
> principal 校长 负责人



###**service** :

> RunService:接口可以运行和配置

####可发射的工作流组合类：

 sliderAppMaster sliderClient,slider--main
做serviceInit要初始化SliderYarnClientImpl加入addService，YARNRegistryClient

####单一类：
- 工作流rpc
- 工作流执行类：队列服务，role服务 调度回调
- 工作流序列类：HBase/SliderAm/agent/....ProviderService
 


####**火箭台**：

- LaunchedApplication：操作SliderYarnClientImpl(流向)，发射

- AbstractLauncher：AppMasterLauncher containerLauncher


- ServiceLauncher :service提供运行的service基础执行者


####提供支持者：

- SliderAMClientProvider 不是service继承自Configured
- AgentClientProvider
- AbstractProviderService
（SliderAMProviderService hbase agenet accumulo etc）





