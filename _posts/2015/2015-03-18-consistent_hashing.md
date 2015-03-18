---
layout: post
title: 一致性hash
categories:
- 逻辑与现象
- CAP
tags:
- 算法
- CAP
---


<i class="icon-file"></i>一致性hash
============

看这个[consistent hashing](http://blog.csdn.net/cywosp/article/details/23397179)

概念：
> BMSL:平衡单调分散负荷

我的概念：

	- 平衡性：数据必须均匀才能使用
	
	- 单调就是：hash后，不能变位置，或落在新位置，不能用已经落的位置。
	
	- 分散：当机器存放的hash区间特别大的时候，有的机器特别小，要避免热点。
	
	- 负荷同分散。

zhaohm:

- HASH链
- 桶
- 建hash表
- 避免冲突链个数放1或控制查找长度
- 复杂度o1
- 成倍扩容数据，分发冲突链，造成全部数据移动
- 传统HASH
- ---------
- HASH环链
- 定义机器链表HASH值
- 将区间放入机器
- 移动机器不会造成全部数据移动