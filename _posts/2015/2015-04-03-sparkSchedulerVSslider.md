---
layout: post
title: spark架构VSslider架构初识
categories:
- 逻辑与现象
- spark
tags:
- spark
- yarn
- slider
---

spark架构VSslider架构初识 -slider关闭spark on yarn with slider jira思考
============


> 
> 最近整理的两个重点中，有yarn中关于resourcescheduler调度的东西，主要是公平调度，rdf多路复用，抢占，treequece,三点资源分配，到此只负责到container分配问题，而真正用户程序如何满足自身调度（操作）程序管理，留给用户实现；另一个重点是关于TaskSchduler能够做到粗粒度的资源的调度管理，给出了一个比较好的用户级的操作策略。结合之前的SliderAmMaster分析，比较其中区别联系。

vs
---------------------


| 功能        | slider           | spark  |
| :------------- |:-------------| :-----|
| am client      | 略 | 略 |
| am  管理ui     | 相同      |   相同 |
| am webapp | 相同     |  相同 |
| am 注册rm | 相同     |  相同 |
| am 注册长时服务 | 支持运行container内部服务注册     |  不支持 |
| am providerservice | 支持各种服务运行的提供者编程     |  不需要，excutor都很简单 |
| am 自定义rpc通信 | 支持am操作服务（比如伸缩，动态伸缩（目前不支持），启停container里的服务）     |  TaskSchduler和DAGschduler负责简单通知启动excutorbackend,不支持手动的excutor的粗粒度整体操作，更不支持更细粒度的操作 |
| am container placement policy | 目前jira更改的支持历史记录判断启停情况和选定机器分配container     |  没有更细的只使用yarn自带的 |
| am anti-annatiy[here](http://jayfans3.github.io/2015/03/sliderJiraContainerPlacement/)) | 把相同compoenent类型可以分离开喜好关联，放到不同的机器上   |  全是excutor不用考虑，把此类excutor放到不同机器上，没有喜好问题 |
| am agent监控 | 每个container有一个监控agent,但只有总的application级compnent-ui控制|  每一个excutor有一个excutorbackend，Ui只有进度条显示,sqlplan中总体的stages,每个stage所用的task-partition |
| am role支持 | 每个角色有不同的component用途，比如启动regionserver,启动storm  | 每个excutor用途只是每个stage有关，stage和执行的sqlplan有关，或者是rdd flunce有关 |
| am delayed队列线程池 | 支持操作和异步动作放入队列等待操作，顺序操作调度     |  fiarpool,和fifopool，采取了和FairScheduler一样的调度？ |
| am 支持调度常驻操作？？|  不支持，只能一次任务一个请求给远程am|  yarnclient支持本地Driver远程WorkLauncher,不能关driver，可操作shell,但不是真正的本地am(负责调度)|
| am 粒度 | 控制的是服务 粗   |  控制的是excutor，里有多个task-partition 粗|




--------------


>  
 上述逻辑需要重新认识。但是可以说明长时间服务运行以及区别出不同服务运行，与短时服务spark的stage区别不同业务的方式有很多差别，有些可以互相借鉴，有些本身就是使用，出现有其意义。以后文章再做讨论。