---
layout: post
title: slider使用的服务及逻辑4(未)
categories:
- 逻辑与现象
- slider 服务设计
tags:
- slider
- 服务
---


<i class="icon-file"></i>slider使用的服务及逻辑(未)
============

 
 接着：[slider概念](http://jayfans3.github.io/2015/03/slider_server/)

 > **我的code:**



####app state

	/**
	 * The model of all the ongoing state of a Slider AM.
	 *
	 * concurrency rules: any method which begins with <i>build</i>
	 * is not synchronized and intended to be used during
	 * initialization.
	 */

####rpcservice代码

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
	 * life of the Java process that f
	 * 
	 * orked it.
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



renewing机制
------------
无



接口多继承的意义
----------
	public interface ProviderService extends ProviderCore,
	    Service,
	    RMOperationHandlerActions,
	    ExitCodeProvider 



概念四
----------

单例队列服务