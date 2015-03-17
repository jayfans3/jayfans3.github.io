---
layout: post
title: slider使用的服务及逻辑
categories:
- 现象与使用
- slider 服务设计
tags:
- slider
- 服务
---


<i class="icon-file"></i>slider使用的服务及逻辑
============

 
 接着：[slider概念](http://jayfans3.github.io/2015/03/slider_server/)

 > **总结一下:**


###工作流组合类（可发射的）：
- sliderAppMaster
- sliderClient
- slider(SliderYarnClientImpl,YARNRegistryClient)
###单一类：
- 工作流rpc
- **工作流执行类**：队列服务，role服务 调度回调
- 工作流序列类：HBase/SliderAm/agent/....ProviderService
###提供支持者：
- SliderAMClientProvider 不是service继承自Configured
- AgentClientProvider
- AbstractProviderService
（SliderAMProviderService hbase agenet accumulo etc）


 
**基本上所有的服务都在此了。底层抽象service具备服务启停抽象，abstractservice发育初始化抽象，它是服务的祖先，所有服务均可init配置。它的孩子有很多服务。组合服务可以包括进其他服务（addService）并依次启动，有可被监听特征，子孙slideram，sliderclient,slider，可以包含其他所有的服务，控制了服务的周期。例如sliderClient内控制了yarnrpc服务和注册服务。sliderappMaster控制了大部分服务。**

 **下面讲下它包含的概念和逻辑。**
为了真实反应am内做的事情，进行了代码分析，摘要代码为：

  首先是启序列服务。


-----------------
	/**
	 * This resembles the YARN CompositeService, except that it
	 * starts one service after another
	 * 
	 * Workflow
	 * <ol>
	 *   <li>When the <code>WorkflowSequenceService</code> instance is
	 *   initialized, it only initializes itself.</li>
	 *   
	 *   <li>When the <code>WorkflowSequenceService</code> instance is
	 *   started, it initializes then starts the first of its children.
	 *   If there are no children, it immediately stops.</li>
	 *   
	 *   <li>When the active child stops, it did not fail, and the parent has not
	 *   stopped -then the next service is initialized and started. If there is no
	 *   remaining child the parent service stops.</li>
	 *   
	 *   <li>If the active child did fail, the parent service notes the exception
	 *   and stops -effectively propagating up the failure.
	 *   </li>
	 * </ol>
	 * 
	 * New service instances MAY be added to a running instance -but no guarantees
	 * can be made as to whether or not they will be run.
	 */
