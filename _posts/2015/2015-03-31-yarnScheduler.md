---
layout: post
title: yarnScheduler使用
categories:
- 逻辑与现象
- yarn服务设计使用
tags:
- yarn
- 调度
---

[TOC]


#yarnScheduler使用
============

##概念
---
- Allow for pluggable execution contexts in resourcemanager
- mr1发展而来，fifo,CapacityScheduler FairScheduler
- yarn.resourcemanager.scheduler.class
- 必须实现resourcescheduler extends YarnScheduler ,YarnScheduler extends EventHanlder<SchduerEvent>
- 在handler.handler()里实现各种操作，通过SchdulerEvent状态机。
- 状态机：

![](/images/2/32.png)


##资源模型

###定义
- yarn.nodemanager.resource.memory-mb 8mb*1024
- yarn.nodemanager.vmem-pmen-ratio 2.1
- yarn.nodemanager.resource.cpu-cores 8
- 虚拟cpu

###能做的事情（语义）
- 某节点某capacity container
- 某rack某capacity container
- blacklist
- cancel container

###不能做的事情（语义）
---
- 任意节点任意机架
- 一组或几组特定资源
- 超细粒度cpu性能，绑定 
- 动态调整container yarn将1.7支持


##调度模型

###概念1 capacity->container

- **drf 多用户多资源调度变为单调度****incremental placement 增量资源分配**采用浪费vs饿死**all or nothing**
- 双层资源调度模型am-container 研究第一层
- pull心跳缓冲
- 心跳状态机

###概念2 queueTree
- 叶子队列提交
- 最少容量比
- 分配给init小的
- 最小容量抢占
- tree抢占默认无限大队列
- 队列映射userusergroup
- ROOT.C1.C12


##调度实现使用：

###capacityScheduler特性配置
- capacity%
- maximum-capacity%
- minimun-user-limit-percent%
- user-limit-factor%
- maximun-applications 10000
- maximun-am-resource-percent0.1
- state= stop running
- yarn mradmin -refreshQueces 

###**fair Scheduler特性配置**
--------
- **drf多路复用**
- 抢占
- 负载均衡
- 为quecetree设置scheduler
- treefile
- preemption
- sizebaseweight
- **locality.threshold.node**
- **lcoality.threshold.rack**
- increment-allocation-mb
- minResources maxResrouces maxRunningApps,minisharePreemptionTimeout
- schedulingMode/policy


##fair scheduler 实现
- NODE_UPDATE/APP_ADDED/APP_REMOVED/NODE_ADDED/NODE_REMOVED/CONTAINER_EXPRIED
- **调度策略决定了叶子队列和非叶子队列是叶间调度还是内部程序调度**
- **三级资源分配策略**
- 选择队列：递归计算其他逻辑算叶子资源量，遇到叶子程序停止
- 选择一个application,id排序就是fair
- **选择一个权重高的container分配，同一优先级container，node local,rack local,nolocal**
- 之后分配完成。

##调度模型抽象
-
![](/images/2/33.png)




##queue->capacity 抢占


- 队列就是trees(use,init)
- 最小资源不是硬资源保证
- 负载转移
- 抢占超过等待时间的队列
- 实现peemtableresourcescheduler
- yarn.resourcemanager.scheduler.monitor.enable=true
- SchedulingEditPolicy yarn.resourcemanager.scheduler.monitor.policies
- SchedulingMonitor inteval触发抢占


###决定抢占

- R=F(min,running+run,父兄节点共享) if u>R then决定抢占

###代价小
-
kill -6 
