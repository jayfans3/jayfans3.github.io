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

###工作流组合类（可发射的）：
- sliderAppMaster
- sliderClient
- slider(SliderYarnClientImpl,YARNRegistryClient)
###单一类：
- 工作流rpc
- **工作流执行服务**：队列服务，role服务 调度回调 fork服务
- **工作流序列服务**：HBase/SliderAm/agent/....ProviderService


 
基本上所有的服务都在此了。底层抽象service具备服务启停抽象，abstractservice发育初始化抽象，它是服务的祖先，所有服务均可init配置。它的孩子有很多。
  
 组合服务可以包括进其他服务（addService）并依次启动，有可被监听特征，子孙slideram，sliderclient,slider，可以包含其他所有的服务，控制了服务的周期。例如**sliderClient内控制了yarnrpc服务和注册服务。sliderappMaster控制了大部分服务**。

 其他的服务，线程类服务加入<E extends ExecutorService>泛型,它的孩子分别是队列服务，role服务，调度服务，分支服务。**队列服务可以添加执行的线程按照顺序执行slider操作，slider使用延迟队列服务来处理抽象slider操作，可延迟特性来自（JDK1.5Delayed）,**

 **下面讲下它包含的概念和逻辑。**
为了真实反应am内做的事情，进行了代码分析，摘要代码为：

  首先是启动两个callbacker服务，在启动slider本身的rpc服务，启动注册服务，启动agentweb,amweb,c1注册，初始化完成agentprovider服务，生成实例，运行role(队列，agent),amp和agentp绑定当前状态和队列，启动操作处理队列，启动amp，启动现象级agentp.


####app state

	/**
	 * The model of all the ongoing state of a Slider AM.
	 *
	 * concurrency rules: any method which begins with <i>build</i>
	 * is not synchronized and intended to be used during
	 * initialization.
	 */

####slider rpc service 

	SliderClusterProtocolPBImpl protobufRelay =
	        new SliderClusterProtocolPBImpl(this);
	    BlockingService blockingService = SliderClusterAPI.SliderClusterProtocolPB
	        .newReflectiveBlockingService(
	            protobufRelay);
	
	    int port = getPortToRequest(instanceDefinition);
	    rpcService =
	        new WorkflowRpcService("SliderRPC", RpcBinder.createProtobufServer(
	            new InetSocketAddress("0.0.0.0", port),
	            getConfig(),
	            secretManager,
	            NUM_RPC_HANDLERS,
	            blockingService,
	            null));
	    deployChildService(rpcService);

####调度服务参考:

	/**
	 * A service that calls the supplied callback when it is started -after the 
	 * given delay. It can be configured to stop itself after the callback has
	 * completed, marking any exception raised as the exception of this service.
	 * The notifications come in on a callback thread -a thread that is only
	 * started in this service's <code>start()</code> operation.
	 */

####这个是workflowsequece

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




forkservice  分支工作流服务
----------------------

	/**
	 * Service wrapper for an external program that is launched and can/will terminate.
	 * This service is notified when the subprocess terminates, and stops itself 
	 * and converts a non-zero exit code into a failure exception.
	 * 
	 * <p>
	 * Key Features:
	 * <ol>
	 *   <li>The property {@link #executionTimeout} can be set to set a limit
	 *   on the duration of a process</li>
	 *   <li>Output is streamed to the output logger provided</li>.
	 *   <li>The most recent lines of output are saved to a linked list</li>.
	 *   <li>A synchronous callback, {@link LongLivedProcessLifecycleEvent}, is raised on the start
	 *   and finish of a process.</li>
	 * </ol>
	 *
	 * Usage:
	 * <p></p>
	 * The service can be built in the constructor, {@link #ForkedProcessService(String, Map, List)},
	 * or have its simple constructor used to instantiate the service, then the 
	 * {@link #build(Map, List)} command used to define the environment variables
	 * and list of commands to execute. One of these two options MUST be exercised
	 * before calling the services's {@link #start()} method.
	 * <p></p>
	 * The forked process is executed in the service's {@link #serviceStart()} method;
	 * if still running when the service is stopped, {@link #serviceStop()} will
	 * attempt to stop it.
	 * <p></p>
	 * 
	 * The service delegates process execution to {@link LongLivedProcess},
	 * receiving callbacks via the {@link LongLivedProcessLifecycleEvent}.
	 * When the service receives a callback notifying that the process has completed,
	 * it calls its {@link #stop()} method. If the error code was non-zero, 
	 * the service is logged as having failed.





queceservice 队列工作流服务
--------------------


	/**
	 * Service wrapper for an external program that is launched and can/will terminate.
	 * This service is notified when the subprocess terminates, and stops itself 
	 * and converts a non-zero exit code into a failure exception.
	 * 
	 * <p>
	 * Key Features:
	 * <ol>
	 *   <li>The property {@link #executionTimeout} can be set to set a limit
	 *   on the duration of a process</li>
	 *   <li>Output is streamed to the output logger provided</li>.
	 *   <li>The most recent lines of output are saved to a linked list</li>.
	 *   <li>A synchronous callback, {@link LongLivedProcessLifecycleEvent}, is raised on the start
	 *   and finish of a process.</li>
	 * </ol>
	 *
	 * Usage:
	 * <p></p>
	 * The service can be built in the constructor, {@link #ForkedProcessService(String, Map, List)},
	 * or have its simple constructor used to instantiate the service, then the 
	 * {@link #build(Map, List)} command used to define the environment variables
	 * and list of commands to execute. One of these two options MUST be exercised
	 * before calling the services's {@link #start()} method.
	 * <p></p>
	 * The forked process is executed in the service's {@link #serviceStart()} method;
	 * if still running when the service is stopped, {@link #serviceStop()} will
	 * attempt to stop it.
	 * <p></p>
	 * 
	 * The service delegates process execution to {@link LongLivedProcess},
	 * receiving callbacks via the {@link LongLivedProcessLifecycleEvent}.
	 * When the service receives a callback notifying that the process has completed,
	 * it calls its {@link #stop()} method. If the error code was non-zero, 
	 * the service is logged as having failed.



----------------

	/**
	 * Execute a long-lived process.
	 *
	 * <p>
	 * Hadoop's {@link org.apache.hadoop.util.Shell} class assumes it is executing
	 * a short lived application; this class allows for the process to run for the
	 * life of the Java process that forked it.
	 * It is designed to be embedded inside a YARN service, though this is not
	 * the sole way that it can be used
	 * <p>
	 * Key Features:
	 * <ol>
	 *   <li>Output is streamed to the output logger provided</li>.
	 *   <li>the input stream is closed as soon as the process starts.</li>
	 *   <li>The most recent lines of output are saved to a linked list</li>.
	 *   <li>A synchronous callback, {@link LongLivedProcessLifecycleEvent}, is raised on the start
	 *   and finish of a process.</li>
	 * </ol>
	 * 
	 */