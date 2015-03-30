---
layout: post
title: slider的管理container逻辑
categories:
- 逻辑与现象
- slider 服务设计
tags:
- slider
- 服务
---


slider的管理container逻辑(未)
============

[https://issues.apache.org/jira/browse/SLIDER-743](https://issues.apache.org/jira/browse/SLIDER-743):

选择位置需要暗示：（包含历史）
slider默认如果超过阙值container起不来的话是放弃本地性数据，启动成功。
但是有些应用是不合理的
                   如果超过阙值container起不来的话是不能放弃本地数据，是要快速失败的。
                   如果超过阙值container起不来的话是
                   检查一下有没有本地数据问题，有就失败，没有就成功。

严格时候就跳过





询问container, 检查history之前role用过的节点，跳过已经实例节点，列表移除这个节点
节点位置有了的时候，它将从pendlist移除，
没有方法记录请求的位置，我们不关心它在哪个节点上，我们记录哪个节点未解决的请求列表
定位到请求的节点算法是：移除未解决请求列表的确定节点
定位到另外的节点算法是：有未解决请求列表不做任何事情
未解决请求列表确定：清除已有位置的未解决请求列表。

当container在新的定位节点失败时候，得有策略。

考虑最好方式解决：1.这节点不能运行app 2.这结点暂时可以考虑来用

我们最终个问题：这结点坏了，至少对这个app

我们造一个黑名单，失败一次加一次暗示给YARN，yarn也有我们没有用。

步骤：
1.严格时候，add container必须问下前一节点（尽管我们要考虑，过去至少在相同的位置启动一次..我不知道前一个定位没有启动离开还是启动的情况。。。）
2.位置喜好时候，如何反应给失败？短暂问题，重试一次，永久问题，重试是错的。


用失败次数记录不对的，不仅失败，还要周期内重置它，这要可以记录组件失败次数。


--------------------------------------------------------------------------




##关于 affinity/anti-affinity 


将相同的label的不同的组件当做不同container请求来处理，比如两个版本的hbase拥有regionserver相同标签，那么都会放入一个
[https://issues.apache.org/jira/browse/YARN-1042](https://issues.apache.org/jira/browse/YARN-1042)

[https://issues.apache.org/jira/browse/SLIDER-82](https://issues.apache.org/jira/browse/SLIDER-82)

[https://issues.apache.org/jira/browse/TWILL-87](https://issues.apache.org/jira/browse/TWILL-87)

There would be a conflit with the YARN configuration that YARN won't return the container on the same node

We remember where nodes where and ask for it back on the same location (though if that request fails, put that location at the end of list of nodes to ask for)

Normally this is a best-effort request; you can set "yarn.placement.policy" to request "strict" placement, in which the AM says the component must be on a specific host. If this cannot be satisfied (As when the host is down), the request remains outstanding. This is for use by applications like Kafka where it is better to have an unsatisifed request than restart elsewhere.

see org.apache.slider.providers.PlacementPolicy



---------------------------------------------

##AM to decide when to relax placement policy from specific host to rack/cluster



[https://issues.apache.org/jira/browse/YARN-3309](https://issues.apache.org/jira/browse/YARN-3309)

[https://issues.apache.org/jira/browse/SLIDER-799](https://issues.apache.org/jira/browse/SLIDER-799)



-----------------------------------------------

slider container jira:
-----
REST操作slidercluster使用 ，thrift


看jira 
[https://issues.apache.org/jira/browse/SLIDER-702](https://issues.apache.org/jira/browse/SLIDER-702) #

[https://issues.apache.org/jira/browse/SLIDER-532](https://issues.apache.org/jira/browse/SLIDER-532)docker



container：
-----------
[https://issues.apache.org/jira/browse/SLIDER-829](https://issues.apache.org/jira/browse/SLIDER-829)当allocated，能取消

[https://issues.apache.org/jira/browse/SPARK-2687](https://issues.apache.org/jira/browse/SPARK-2687)


[https://issues.apache.org/jira/browse/SLIDER-743](https://issues.apache.org/jira/browse/SLIDER-743) 

###Add co-processor support for app packages

[https://issues.apache.org/jira/browse/SLIDER-799](https://issues.apache.org/jira/browse/SLIDER-799)

[https://issues.apache.org/jira/browse/SLIDER-726](https://issues.apache.org/jira/browse/SLIDER-726) container

[https://issues.apache.org/jira/browse/SLIDER-629](https://issues.apache.org/jira/browse/SLIDER-629) #


https://issues.apache.org/jira/browse/SLIDER-764

https://issues.apache.org/jira/browse/SLIDER-787

https://issues.apache.org/jira/browse/SLIDER-629

https://issues.apache.org/jira/browse/SLIDER-600

https://issues.apache.org/jira/browse/SPARK-3561


https://issues.apache.org/jira/browse/SLIDER-825


https://issues.apache.org/jira/browse/SLIDER-773

https://issues.apache.org/jira/browse/SLIDER-611




------------
https://issues.apache.org/jira/browse/SLIDER-691

https://issues.apache.org/jira/browse/SLIDER-695



