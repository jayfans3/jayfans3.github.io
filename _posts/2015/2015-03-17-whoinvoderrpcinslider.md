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

客户端得到代理，使用协议
	RpcBinder：
	
	 public static SliderClusterProtocol connectToServer(InetSocketAddress addr,
	                                                    UserGroupInformation currentUser,
	                                                    Configuration conf,
	                                                    int rpcTimeout) throws IOException {
	    Class<SliderClusterProtocolPB> sliderClusterAPIClass = registerSliderAPI(
	        conf);
	
	    log.debug("Connecting to Slider AM at {}", addr);
	    ProtocolProxy<SliderClusterProtocolPB> protoProxy =
	      RPC.getProtocolProxy(sliderClusterAPIClass,
	                           1,
	                           addr,
	                           currentUser,
	                           conf,
	                           NetUtils.getDefaultSocketFactory(conf),
	                           rpcTimeout,
	                           null);
	    SliderClusterProtocolPB endpoint = protoProxy.getProxy();
	    return new SliderClusterProtocolProxy(endpoint);
	  }
	  liderClusterProtocolProxy implements SliderClusterProtocol ,代理帮助客户端来操作SliderClusterProtocolPB endpoint句柄，通过PB来取得结果
	
	
	getProtocolEngine(protocol,conf).getProxy(protocol, clientVersion,
	        addr, ticket, conf, factory, rpcTimeout, connectionRetryPolicy);
	


具体的SERVICE端调用一样会使用


