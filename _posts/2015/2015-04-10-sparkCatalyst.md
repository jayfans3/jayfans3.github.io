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

- unresolvedrelation 意思就是还没有bind，等待绑定
- LogicalPlan继承自QueryPlan，内部带一个reference:Set[Attribute]，主要方法为resolve(name:String): Option[NamedeExpression]，用于分析生成对应的NamedExpression
- LeafNode-DDLcommand: set,describe,cache,native
- UnaryNode:sort,distinct,aggregate,filter,limit,subquery
- binaryNode:join,union

###physical plan:SparkPlan具体系统里实现

- leafNode:exist,dscribe,parquet,set
- unaryNode:sort,genarate,exchange,fifter,aggregate,order
- binaryNode：broadcast,cartesian,hashjoin,leftsemijoinBNL,leftsemijoinHash
- 分区表示模型

###plan映射QueryPlanner:具体系统里实现

- QueryPlanner[Physical <: TreeNode[PhysicalPlan]]
- SparkStrategies Seq[Strategy].apply -|> queryplanner
- 内部制定LeftSemiJoin, HashJoin,PartialAggregation, BroadcastNestedLoopJoin,CartesianProduct等几种策略
- 每种策略接LogicalPlan，生成Seq[SparkPlan]，每个SparkPlan-具体RDD的算子

###expression:指不需要plan计算和处理node

- Cast，Projection，四则运算，逻辑操作符

###Rules:规则匹配/节点处理.|>RuleExecutor[TreeType]

- protected case class Batch(name: String, strategy: Strategy, rules: Rule[TreeType]*)  
- hql数据集来自hive api (parquet json 是SQLContext)



###sequence1
-

->catalystparser->catalystanalyzer->catalystoptimizer->sparkplanner,sparkstrategy->sqlplan.execute

###sequence2
-

> ->parseSql(sql:string):LogicalPlan

> 初步映射SimpleCatalog：**new Analyzer(catalog, EmptyFunctionRegistry, caseSensitive =true)不支持udf**，简单batchs.rule实现MultiInstanceRelations,CaseInsensitiveAttributeReferences,Resolution,Check Analysis,AnalysisOperators)

> ->optimizer:batchs.rule:Combine Limits,ConstantFolding,Filter Pushdown)

> ->SparkPlanner:

		01 val strategies: Seq[Strategy] =  
		02.  CommandStrategy(self) ::  
		03.  TakeOrdered ::  
		04.  PartialAggregation ::  
		05.  LeftSemiJoin ::  
		06.  HashJoin ::  
		07.  InMemoryScans ::  
		08.  ParquetOperations ::  
		09.  BasicOperators ::  
		10.  CartesianProduct ::  
		11.  BroadcastNestedLoopJoin :: Nil  


> ->prepareForExecution :

	01.val batches =  
	02.  Batch("Add exchange", Once, AddExchange(self)) ::  
	03.  Batch("Prepare Expressions", Once, new BindReferences[SparkPlan]) :: Nil  


> ->execute递归tree


###数据源
TextInputFormat:key class是LongWritable:value class是Text，value=RDD[String]
JsonRDD.inferSchema(RDD[String])