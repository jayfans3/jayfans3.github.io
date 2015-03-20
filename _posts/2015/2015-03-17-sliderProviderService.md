---
layout: post
title: slider使用的服务及逻辑3-序列服务
categories:
- 逻辑与现象
- slider 服务设计
tags:
- slider
- 服务
---


<i class="icon-file"></i>slider使用的服务及逻辑
============

 
 接着：[slider概念](http://jayfans3.github.io/2015/03/slider_server/)

 > **我的code:**
RMOperationHandlerActions

容器分配时候运行role方法，role用来把syscaction的startcontainer放入立即队列，并history.

onallocated帮你启动container吗？不管。只管AMcontainer回调的那些。containerprovier完成配置。

appstate：发送roleinstance，app是component。


roleservice 启动container之后做了。
-----
 /**
   * Note that the master node has been launched,
   * though it isn't considered live until any forked
   * processes are running. It is NOT registered with
   * the role history -the container is incomplete
   * and it will just cause confusion
   */
  public void noteAMLaunched() {
    getLiveNodes().put(appMasterNode.getContainerId(), appMasterNode);
  }


provider
==================================

最后启动两个provoder服务：

 // Start the Slider AM provider
    sliderAMProvider.start();

    // launch the real provider; this is expected to trigger a callback that
    // starts the node review process
    launchProviderService(instanceDefinition, confDir);


