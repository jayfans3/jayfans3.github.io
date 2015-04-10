---
layout: post
title: spark Catalyst
categories:
- 逻辑与现象
- spark
tags:
- spark
- sql
- sparksql
---

Catalyst
============

##TreeNode范型

- 内存
- 逻辑执行还是优化都是替换结点
- children: Seq[BaseType].foreach、map、collect
- transformDown,transformUp
- UnaryNode,BinaryNode, LeafNode三种trait：非叶子节点允许有一个或两个子节点
- 