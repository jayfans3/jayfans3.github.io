---
layout: post
title: phoenix数据导入
categories:
- 人
tags:
- 书
---



#phoenix数据导入

使用参考：

测试步骤和环境：


环境：
1.集群需要：zk，hadoop,hbase
2.需要测试稽核的数据：目录为app/bulkload/JL.DAT(此数据是tsv格式，数据按TAB键分开的行)，上传到hdfs

3.如果是TAB,默认phoenix支持单字符，需要修改源码：

jar:phoenix-core 中的org.apache.phoenix.mapreduce.CsvBulkLoadTool.java char delimiterChar = ',' 修改你想要的双字符，注意是"\t",需要重新编译，命令为：

	mvn clean package -DskipTests -Dhbase.version=1.0.0 -Dhadoop.version=2.5.0-cdh5.2.1

如果是cdh版，需要maven添加：

	<repository>
	<id>cloudera</id> 
	<url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
	</repository>

测试步骤：

1. 启动hbase：
检查hbase/conf目录下backmaster配置为集群上另外一台安装了hbase的机器hostname；
检查hdfs-site.xml和core-site.xml指定hadoop环境（自行配置）；
检查hbase/conf目录下regionserver配置，regionserver指定regionserver启动目录；
检查hbase/conf目录下hbase-env.sh配置HBASE所需的JVM，REGIONSERVER内存，zk管理；
检查hbase/lib下是否有phoenix-client版本包。
分发启动。

2. 使用phoenix客户端连接hbase：
启动hbase后，页面可以看到系统表中加入coperssor为phoenix相关类，即可证明安装成功；
使用phoenix对应版本的客户端，将hbae-site.xml放入phoenix/bin目录；
在phoenix/bin目录下运行shell: ./sqlline.py ochadoop400:2184；
可用sql进行查询，upsert into select count(*)，where等等具体语法参考官方文档。

3. 使用phoenix建表：
create table CON_DOMOB_API(USER_ID varchar not null,TAG_INDEX varchar,TIME varchar not null,DAY_ID varchar , FLAG varchar not null CONSTRAINT PK PRIMARY KEY (USER_ID,TIME,FLAG)) salt_buckets=10;
例子为 创建一个phoenix的hbase映射表，分区为10. 字段一律大写，主键是联合主键。

4. 使用load.sh 进行phoenix bulkload插入hbase数据
首先将ocnosql-commonjar放入hadoop/share/hadoop/yanr/lib下，
	3. 将phoenix-client放入regionserver.lib restart 分发

最终执行语句为
hadoop jar $jarPath $loadType -Dmapreduce.map.java.opts="-server -Xms3072m -Xmx3072m -XX:NewSize=2400m -XX:MaxNewSize=2400m -XX:PermSize=128m -XX:MaxPermSize=128m -XX:ParallelGCThreads=2" -Dmapreduce.map.memory.mb="4096" -Dmapreduce.reduce.java.opts="-server -Xms3072m -Xmx3072m -XX:NewSize=2400m -XX:MaxNewSize=2400m -XX:PermSize=128m -XX:MaxPermSize=128m -XX:ParallelGCThreads=2" -Dmapreduce.reduce.memory.mb="4096" -t $tableName -i $inputPath -o $outputPath  -c $columns -z "dmp001:2187"

讲环境中的jar加入，env、hbase目录、hadoop目录配好，删除hdfs上生成的mapreduce目录文件
执行：
sh phoenix_load_original.sh USE6TABLE /dmptest/user/liujs/a > TEST.LOG 2>&1 &

4.     观察数据导入使用select count(*) from table;至此phoenixbulload完成。


目前ocnosql工作：1.query:api，filter,单列，多列，sql,数据源 2.rowkey,regioncreate


注：

Installation
To install a pre-built phoenix, use these directions:

Download and expand the latest phoenix-[version]-bin.tar.
Add the phoenix-[version]-server.jar to the classpath of all HBase region server and master and remove any previous version. An easy way to do this is to copy it into the HBase lib directory (use phoenix-core-[version].jar for Phoenix 3.x)
Restart HBase.
Add the phoenix-[version]-client.jar to the classpath of any Phoenix client.