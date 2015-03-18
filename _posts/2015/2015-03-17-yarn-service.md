---
layout: post
title: yarn中服务和事件库的使用
categories:
- 逻辑与现象
- yarn服务设计使用
tags:
- 服务
- rpc
---


##yarn中服务和事件库的使用

-class abstractevent<>  夹括号意义
  创建类型作为参数的类

a[m.blog.csdn.net/blog/microchenhong_11109/6239462]

一个服务：接口，包括四个状态，没初始化，初始化，启动，停止，有listener可观察
实现上层服务，有单一，组合，slider中出现的有很多。

- 服务事件是本地概念 和远程调用无关
- yarn 核心服务am rn nn 一个dispathcer

- 使用 服务来派发事件类型给不同的事件impl处理器。使用代码实现。

- 异步并发 对象抽象成事件处理器

- 事件通知对象状态判断结束。

------------
疑问 dispathcher 实现 和通知结束 服务和派发的联系 为什么要启动service实现是什么？ 服务库是不是监听者模式如何实现  注册的东西什么？
 


回调实现
编程库 client am nm 编程
粗力度
