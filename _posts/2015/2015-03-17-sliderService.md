---
layout: post
title: slider使用的服务及逻辑
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

 > **我的逻辑:**

> 工作流：main序列图

###工作流组合服务（可发射的）：
- sliderAppMaster
- sliderClient
- slider(SliderYarnClientImpl,YARNRegistryClient)
###工作流单一服务：
- **工作流rpc服务** ： slider 操作的服务器端[server代码](#server代码)
- **工作流线程池服务**：队列池服务，role池服务，调度回调池 fork池服务
- **工作流序列服务**：HBase/SliderAm/agent/....ProviderService


 
基本上所有的服务都在此了。底层抽象service具备服务启停抽象，abstractservice发育初始化抽象，它是服务的祖先，所有服务均可init配置。它的孩子有很多。
  
 组合服务可以包括进其他服务（addService）并依次启动，有可被监听特征，子孙slideram，sliderclient,slider，可以包含其他所有的服务，控制了服务的周期。例如**sliderClient内控制了yarnrpc服务和注册服务。sliderappMaster控制了大部分服务**。

 其他的服务，线程池服务加入<E extends ExecutorService>泛型,它的孩子分别是队列服务，role服务，调度服务，分支服务。序列服务是伴随着container周期顺序执行具体实现的服务。**

 **序列：**


  首先是启动两个callbacker服务，在启动slider本身的rpc服务，启动注册服务，启动agentweb,amweb,c1注册，初始化完成agentprovider服务，生成实例，运行role服务(队列，agent),amproviderservice和agentproviderservice绑定当前状态和队列，启动操作处理队列，启动amproviderservice，启动实际的agentproviderservice.



####这个是sliderammaster主要的逻辑

----------------

	// initAndAddService(providerService);
	
	//sliderAMProvider = new SliderAMProviderService();
	//yarnRPC = YarnRPC.create(serviceConf);
	// asyncRMClient = AMRMClientAsync.createAMRMClientAsync(heartbeatInterval, this);
	//nmClientAsync = new NMClientAsyncImpl("nmclient", this);
	//startSliderRPCServer();
	//startRegistrationService();
	// startAgentWebApp(appInformation, serviceConf);
	//webApp = new SliderAMWebApp(registry);
	//new WebAppService<SliderAMWebApp>("slider", webApp);
	//appState.buildInstance(instanceDefinition,
	          serviceConf,
	          providerConf,
	          providerRoles,
	          fs.getFileSystem(),
	          historyDir,
	          liveContainers,
	          appInformation,
	          new SimpleReleaseSelector());
	//launchService = new RoleLaunchService(actionQueues,
	                                          providerService,
	                                          fs,
	                                          new Path(getGeneratedConfDir()),
	                                          envVars,
	                                          launcherTmpDirPath);
	// sliderAMProvider.start();
	    // launch the real provider; this is expected to trigger a callback that
	    // starts the node review process
	//launchProviderService(instanceDefinition, confDir);providerService start



