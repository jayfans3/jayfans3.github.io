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


#TreeNode范型

- 内存
- 逻辑执行还是优化都是替换结点
- children: Seq[BaseType].foreach、map、collect
- transformDown,transformUp
- UnaryNode,BinaryNode, LeafNode三种trait：非叶子节点允许有一个或两个子节点

##plan

###LogicalPlan

- LogicalPlan继承自QueryPlan，内部带一个reference:Set[Attribute]，主要方法为resolve(name:String): Option[NamedeExpression]，用于分析生成对应的NamedExpression
- LeafNode-DDLcommand: set,describe,cache,native
- UnaryNode:sort,distinct,aggregate,filter,limit,subquery
- binaryNode:join,union

###physical plan:SparkPlan具体系统里实现

- leafNode:exist,dscribe,parquet,set
- unaryNode:sort,genarate,exchange,fifter,aggregate,order
- binaryNode：broadcast,cartesian,hashjoin,leftsemijoinBNL,leftsemijoinHash
- 分区表示模型

###plan映射:具体系统里实现

- QueryPlanner[Physical <: TreeNode[PhysicalPlan]]
- SparkStrategies Seq[Strategy].apply -|> queryplanner
- 内部制定LeftSemiJoin, HashJoin,PartialAggregation, BroadcastNestedLoopJoin,CartesianProduct等几种策略
- 每种策略接LogicalPlan，生成Seq[SparkPlan]，每个SparkPlan-具体RDD的算子

###expression:指不需要执行引擎计算，而可以直接计算或处理的节点

- Cast，Projection，四则运算，逻辑操作符

###Rules:实施规则匹配和节点处理的，都需要继承RuleExecutor[TreeType]抽象类

- protected case class Batch(name: String, strategy: Strategy, rules: Rule[TreeType]*)  
- hql数据集来自hive api (parquet json 是SQLContext)



sequence1
-

catalystparser->catalystanalyzer->catalystoptimizer->sparkplanner,sparkstrategy->sqlplan.execute

sequence2
-

