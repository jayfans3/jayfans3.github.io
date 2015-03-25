---
layout: post
title: slider版本历史
categories:
- 逻辑与现象
- slider 使用
tags:
- slider
- 服务
---


slider 版本(未)
============
slider版本重要升级使用：

0.50：
0.50 动态扩展的功能update命令 0.50 https://issues.apache.org/jira/browse/SLIDER-223

0.60：

	* [SLIDER-167] - Automatically copy slider-agent.tar.gz when creating an application
	* [SLIDER-105] - Agent provider service/client should reject component instance count when it exceeds the specified maximum or below the minimum


	* [SLIDER-243] - slider client should read hadoop cluster configuration from *-site.xml
	* [SLIDER-149] - Support a YARN service registry
	* [SLIDER-595] - Default app master container size should be 1024m
	* [SLIDER-105] - Agent provider service/client should reject component instance count when it exceeds the specified maximum or below the minimum


0.70：
 support configurating HBASE_OPTS
删除agent tar.gz 放入lib中

	* [SLIDER-743] - Include node failure history when choosing placement hints


