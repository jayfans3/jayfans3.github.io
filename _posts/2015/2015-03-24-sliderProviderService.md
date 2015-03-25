---
layout: post
title: slider使用的服务及逻辑3-序列服务(未)
categories:
- 逻辑与现象
- slider 服务设计
tags:
- slider
- 服务
---


序列服务(未)
============

概念：
record

RegistryOperations


> 
	yarn序列：
	addContainerRequest ->amrm
	startContainerAsync -> nmrm startContainer rolelaunceer
	通过集群操作来和am交互添加container，
	当分配allcocated时候启动role服务，和添加agentpy命令，可以监控。



> application 配置结束后，启动队列和amprovider,privoderservcie,后序列为：
> 
>  flexcluster->reviewRequestAndReleaseNode
>  在onContainersCompleted（）->reviewRequestAndReleaseNodes（）->放入队列服务->最终它检查rolestatus一个个的查看是否改变，改变了就ContainerAssignment（派发container）

下面是关键的代码
-------------
	  /**
	   * Look at where the current node state is -and whether it should be changed
	   */
	  public synchronized List<AbstractRMOperation> reviewRequestAndReleaseNodes()
	      throws SliderInternalStateException, TriggerClusterTeardownException {
	    log.debug("in reviewRequestAndReleaseNodes()");
	    List<AbstractRMOperation> allOperations = new ArrayList<AbstractRMOperation>();
	    for (RoleStatus roleStatus : getRoleStatusMap().values()) {
	      if (!roleStatus.getExcludeFromFlexing()) {
	        List<AbstractRMOperation> operations = reviewOneRole(roleStatus);
	        allOperations.addAll(operations);
	      }
	    }
	    return allOperations;
  
	/**
	   * Look at the allocation status of one role, and trigger add/release
	   * actions if the number of desired role instances doesnt equal 
	   * (actual+pending)
	   * @param role role
	   * @return a list of operations
	   * @throws SliderInternalStateException if the operation reveals that
	   * the internal state of the application is inconsistent.
	   */
	  @SuppressWarnings("SynchronizationOnLocalVariableOrMethodParameter")
	  private List<AbstractRMOperation> reviewOneRole(RoleStatus role)


### 在onContainersAllocated():
### launchService.launchRole()->startcontainer
### 例如：Assigning role
###HBASE_REGIONSERVER to container_1417485373647_0001_01_000002
    RoleLaunchService启动
	Starting container with command: python ./infra/agent/slider-agent/agent/main.py --label container_1417485373647_0001_01_000002___HBASE_MASTER --zk-quorum ocean00:2383 --zk-reg-path /registry/users/ocean/services/org-apache-slider/s6_1 > <LOG_DIR>/slider-agent.out 2>&1



hearbeat ->agentservice->ws->http->调用agentservice返回response.


---
	/* =================================================================== */
	/* AMRMClientAsync callbacks */
	/* =================================================================== */
	
	  /**
	   * Callback event when a container is allocated.
	   * 
	   * The app state is updated with the allocation, and builds up a list
	   * of assignments and RM opreations. The assignments are 
	   * handed off into the pool of service launchers to asynchronously schedule
	   * container launch operations.
	   * 
	   * The operations are run in sequence; they are expected to be 0 or more
	   * release operations (to handle over-allocations)
	   * 
	   * @param allocatedContainers list of containers that are now ready to be
	   * given work.
	   */
	  @SuppressWarnings("SynchronizationOnLocalVariableOrMethodParameter")
	  @Override //AMRMClientAsync
	  public void onContainersAllocated(List<Container> allocatedContainers) {}

------------


	2014-12-01 21:05:40,935 [main] INFO  appmaster.SliderAppMaster - Queue Processing started
	2014-12-01 21:05:40,939 [AmExecutor-005] INFO  actions.QueueService - QueueService processor started
	2014-12-01 21:05:40,950 [AmExecutor-006] INFO  actions.QueueExecutor - Queue Executor run() started
	2014-12-01 21:05:41,485 [main] INFO  agent.AgentClientProvider - Validating app definition hdfs://OCNoSQLBJ/user/ocean/.slider/package/HBASE/slider-hbase-app-package-0.60.0-incubating.zip
	2014-12-01 21:05:41,923 [AmExecutor-006] INFO  state.AppState - Reviewing RoleStatus{name='HBASE_REGIONSERVER', key=2, minimum=0, maximum=1, desired=1, actual=0, requested=0, releasing=0, failed=0, started=0, startFailed=0, completed=0, failureMessage=''} : expected 1
	2014-12-01 21:05:41,923 [AmExecutor-006] INFO  state.AppState - HBASE_REGIONSERVER: Asking for 1 more nodes(s) for a total of 1
	2014-12-01 21:05:41,967 [AmExecutor-006] INFO  state.AppState - Container ask is Capability[<memory:256, vCores:1>]Priority[1073741826] and label = null
	2014-12-01 21:05:41,975 [AmExecutor-006] INFO  state.AppState - Reviewing RoleStatus{name='HBASE_MASTER', key=1, minimum=0, maximum=1, desired=1, actual=0, requested=0, releasing=0, failed=0, started=0, startFailed=0, completed=0, failureMessage=''} : expected 1
	2014-12-01 21:05:41,979 [AmExecutor-006] INFO  state.AppState - HBASE_MASTER: Asking for 1 more nodes(s) for a total of 1
	2014-12-01 21:05:41,980 [AmExecutor-006] INFO  state.AppState - Container ask is Capability[<memory:256, vCores:1>]Priority[1073741825] and label = null
	2014-12-01 21:05:43,669 [AMRM Heartbeater thread] INFO  impl.AMRMClientImpl - Received new token for : ocean01:53021
	2014-12-01 21:05:43,670 [AMRM Heartbeater thread] INFO  impl.AMRMClientImpl - Received new token for : ocean00:60682
	2014-12-01 21:05:43,671 [AMRM Callback Handler Thread] INFO  appmaster.SliderAppMaster - onContainersAllocated(2)
	2014-12-01 21:05:43,677 [AMRM Callback Handler Thread] INFO  state.AppState - Assigning role HBASE_MASTER to container container_1417485373647_0001_01_000002, on ocean01:53021,
	2014-12-01 21:05:43,685 [AMRM Callback Handler Thread] INFO  state.AppState - Assigning role HBASE_REGIONSERVER to container container_1417485373647_0001_01_000003, on ocean00:60682,
	2014-12-01 21:05:43,696 [AMRM Callback Handler Thread] INFO  appmaster.SliderAppMaster - Diagnostics: RoleStatus{name='slider-appmaster', key=0, minimum=0, maximum=1, desired=1, actual=1, requested=0, releasing=0, failed=0, started=1, startFailed=0, completed=0, failureMessage=''}
	RoleStatus{name='HBASE_REGIONSERVER', key=2, minimum=0, maximum=1, desired=1, actual=1, requested=0, releasing=0, failed=0, started=0, startFailed=0, completed=0, failureMessage=''}
	RoleStatus{name='HBASE_MASTER', key=1, minimum=0, maximum=1, desired=1, actual=1, requested=0, releasing=0, failed=0, started=0, startFailed=0, completed=0, failureMessage=''}
	
	2014-12-01 21:05:43,776 [RoleLaunchService-007] INFO  agent.AgentProviderService - Build launch context for Agent
	2014-12-01 21:05:43,780 [RoleLaunchService-008] INFO  agent.AgentProviderService - Build launch context for Agent
	2014-12-01 21:05:43,804 [RoleLaunchService-007] INFO  agent.AgentProviderService - AGENT_WORK_ROOT set to $PWD
	2014-12-01 21:05:43,804 [RoleLaunchService-007] INFO  agent.AgentProviderService - AGENT_LOG_ROOT set to <LOG_DIR>
	2014-12-01 21:05:43,804 [RoleLaunchService-007] INFO  agent.AgentProviderService - PYTHONPATH set to ./infra/agent/slider-agent/
	2014-12-01 21:05:43,807 [RoleLaunchService-008] INFO  agent.AgentProviderService - AGENT_WORK_ROOT set to $PWD
	2014-12-01 21:05:43,807 [RoleLaunchService-008] INFO  agent.AgentProviderService - AGENT_LOG_ROOT set to <LOG_DIR>
	2014-12-01 21:05:43,807 [RoleLaunchService-008] INFO  agent.AgentProviderService - PYTHONPATH set to ./infra/agent/slider-agent/
	2014-12-01 21:05:43,906 [RoleLaunchService-007] INFO  agent.AgentProviderService - Using ./infra/agent/slider-agent/agent/main.py for agent.
	2014-12-01 21:05:43,906 [RoleLaunchService-008] INFO  agent.AgentProviderService - Using ./infra/agent/slider-agent/agent/main.py for agent.
	2014-12-01 21:05:43,975 [RoleLaunchService-007] INFO  appmaster.RoleLaunchService - Starting container with command: python ./infra/agent/slider-agent/agent/main.py --label container_1417485373647_0001_01_000002___HBASE_MASTER --zk-quorum ocean00:2383 --zk-reg-path /registry/users/ocean/services/org-apache-slider/s6_1 > <LOG_DIR>/slider-agent.out 2>&1 ;
	2014-12-01 21:05:43,975 [RoleLaunchService-007] INFO  appmaster.RoleLaunchService - Container launch delay for HBASE_MASTER set to 0 seconds
  
appstate：发送roleinstance，app是component。





