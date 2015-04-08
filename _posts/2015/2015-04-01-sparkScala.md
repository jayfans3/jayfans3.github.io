---
layout: post
title: scala使用
categories:
- 逻辑与现象
- spark
tags:
- spark
- scala 
---


 spark-scala
============

java

	1. 设计模式
	2. 抽象
	3. 分层概念
	4. aop干得行为事情勉强
	5. 还在吗？


scala

	1. sacala对人行为编程
	2. 简化了aop
	3. 引入as js 必包原型连
	4. 泛化实体为数学
	5. 为什么要嵌入java，是因为用java弥补物缺陷比如服务设计，就像java用设计模式和aop弥补行为一样。




scala:
- 由物转人 原来研究由人用物
 
- map
- flatmap
- 没有++
- 省略{}
- 省略;
- 变量:类型 = 值 as语法像
- 单例取代静态
- _ * 
- 闭包
- 特征
- 多继承      
- unit
- 混合
- aspect编程
- 肯定是方面aspect 这样实际的方面来源于特征，需要理解真正的JAVA对比？



以下转载 ：

Scala的类型体系（基本类型）
------------

![基本类型](/images/1/0.png)
Scala基本类型相对应的类

对于整数对象7可以响应各种消息（方法）。既可以执行toString方法来转换成字符串，又可以使用to这个执行Int => Range的方法。附带说一下，7 to 20相当于7.to(20)，该方法的执行结果是Range(7,8, 9, … 19, 20)。对于该范围对象适用了foreach( (i)=>print(i) )，print _则与一个参数的匿名函数(i) => print(i)相当。

	1. scala> 7.toString  
	2. res2: java.lang.String = 7 
	3. scala> 7 to 20 foreach( print _ )  
	4. 7891011121314151617181920 

实际上，Scala在编译器自动引入的Predef单例对象中定义了为了兼容Java基础类型所存在的类型别名。例如boolean, char, byte, short, int, long, float, double被定义了，这些别名实际上是引用了Scala.Boolean,Scala.Char,Scala.Byte等Scala的类。可能的话，为了提高“Scala中说所有数据都是对象”这种意识，建议尽量一开始就使用Int、Boolean、Float等原来的类名。

不过，在Scala种并没有类型转换操作符，而是在所有类的基类Any中定义了具有同等功能的方法asInstanceOf[X]。用这方法就可以把类型转换为X了。Any类中同时还定义了相当于instanceof操作符的isInstanceOf[X]方法。

![基本类型](/images/1/1.png)
特别是该类层次中Iterable下的集类型在函数式编程中大显身手。其中的可变(mutable)与非可变（immutable）两大系列的类层次基本上呈现出镜像关系，可以充分发挥出函数式语言功能的当然就是非可变集类型群了。

