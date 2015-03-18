---
layout: post
title: 编写yarn程序步骤
categories:
- 逻辑与现象
- yarn服务设计使用
tags:
- yarn
- 服务
- rpc
---


编写yarn程序步骤
--------------

###client:

yarnrpcclient对象

- 申请id，
- 查看资源，
- 设置am的containcontext
   (**command localresource env**)
   sumbitcontext发送 
   (**队列权重资源**）
- 监控report


###master:

> main:
> 
- 接参：（Environment，system）
- 两callback对象（**nmcallbacker ,rmcallbacker**）
- (自定义rpcserver启动 注册)
- amresponse算内存 核
- 算containeraddContainerRequest

--------------
###运行自己的cantainer：

   rmcall中命令nmcall运行container:

onContainersAllocated->
LaunchContainerRunnable->
run->
> 
ContainerLaunchContext ctx 
= ContainerLaunchContext.newInstance(
              localResources, shellEnv, commands, null, allTokens.duplicate(), null);
      containerListener.addContainer(container.getId(), container);
      nmClientAsync.startContainerAsync(container, ctx);



