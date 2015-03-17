---
layout: post
title: slider-0.40安装
categories:
- 使用
tags:
- slider
- 安装
---

Apache Slider 0.40 安装文档
=====
######BDX-刘峻山


##系统需求
###最低版本jdk1.7
###	操作系统
> 64-bit Red Hat Enterprise Linux (RHEL) 6
64-bit CentOS 6
64-bit Oracle Linux 6
Hadoop 2.4
HDFS, YARN and ZooKeeper
Python 2.6
Openssl

##安装slider
###编译及准备

编译slider需要protoc协议，安装好protoc后：http://slider.incubator.apache.org/downloads/下载发布版，进行编译。
执行命令：
1.	mvn clean site:site site:stage package -DskipTests
2.	maven设置:set MAVEN_OPTS=-Xms256m -Xmx512m -Djava.awt.headless=true
2.	将编好的 target下slider-0.40.0-incubating-all.tar.gz包解压。
3.	每一台的集群机器上要安装openssl：版本号为：openssl-devel-1.0.1e-30.el6_6.4.x86_64。
root用户下使用yum install openssl-devel –y命令。
4.	机器上每一台安装python 2.6+版本。
### 修改配置

修改配置文件slider-0.40/conf/slider-client.xml，添加配置：

	<property>
	<name>slider.zookeeper.quorum</name>
	<value>{your zk}</value>
	</property>

	<property>
	<name>hadoop.registry.zk.quorum</name>
	<value>yourZooKeeperHost:port</value>
	</property>

	<property>
	<name>yarn.resourcemanager.address</name>
	<value>yourResourceManagerHost:8050</value>
	</property>

	<property>
	<name>yarn.resourcemanager.scheduler.address</name>
	<value>yourResourceManagerHost:8030</value>
	</property>

	<property>
	<name>fs.defaultFS</name>
	<value>hdfs://yourNameNodeHost:8020</value>
	</property>

若是配置了HA添加hdfs-site.xml（假设host为OCNoSQLBJ）：

	<property>
	    <name>slider.yarn.queue</name>
	    <value>default</value>
	    <description>YARN queue for the Application Master</description>
	  </property>
	  
	  <property>
	  <name>dfs.nameservices</name>
	  <value>OCNoSQLBJ</value>

	  <property>
	  <name>dfs.ha.namenodes.OCNoSQLBJ</name>
	  <value>nn1,nn2</value>
	  </property>
	  
	  <property>
	  <name>dfs.namenode.rpc-address.OCNoSQLBJ.nn1</name>
	  <value>ocean00:8020</value>

	<property>
	<name>dfs.namenode.http-address.OCNoSQLBJ.nn2</name>
	<value>ocean01:18083</value>
	</property>
	
	<property>
	<name>dfs.namenode.shared.edits.dir</name>
	<value>qjournal://ocean00:8338;ocean01:8338/OCNoSQLBJ</value>

	<property>
	<name>dfs.client.failover.proxy.provider.OCNoSQLBJ</name>
	<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	
	<property>
	<name>dfs.ha.automatic-failover.enabled</name>
	<value>true</value>

到slider-client.xml中。
用./slider version 命令测试slider安装是否正确。

####在slider上运行hbase

####建立hdfs文件镜像
######slider需要上传到hdfs，执行下列4个命令：
hadoop fs -mkdir -p /user/{your username} /slider/agent/conf
hadoop fs -chown 你的用户 /user/ocean /slider/agent/conf
hadoop fs -put slider-0.40/agent/slider-agent.tar.gz /slider/agent/
hadoop fs -put slider-0.40/agent/agent.ini /slider/agent/conf/

#####制作hbase0983s4.zip并上传hdfs
- 
添加hbasetar包到slider源码打包环境，可以去hbase官网下载，地址：http://archive.apache.org/dist/hbase/hbase-0.98.3/hbase-0.98.3-hadoop1-bin.tar.gz

- 
拷贝 ~/Downloads/hbase-0.98.3-hadoop2-bin.tar.gz 到slider源码的app-package目录的hbase/package/files下

- 
打开appConfig.json/metainfo.xml
代替4个 "${hbase.version}" 用"0.98.3-hadoop2"
代替1个 "${app.package.name}"用"hbase-v098"

- 
压缩并上传
压缩zip -r hbase-v098.zip .
验证压缩zip -Tv hbase0983s4.zip
hadoop fs -put hbase0983s4.zip /slider/

####设置本次启动配置

拷贝app-package中的appConfig.json resources.json  到指定目录下。
编辑appConfig.json修改配置项：
 "application.def":"/slider/hbase0983s4.zip",
 "java_home":"your jdk",
 "package_list":"files/hbase-0.98.3-hadoop2-bin.tar.gz",
 "app_root":"${AGENT_WORK_ROOT}/app/install/hbase-0.98.3-hadoop2",
 "site.hbase_env.hbase_managers_zk":"false",
 "site.global.app_user":"yourname",
 "site.global.user_group":"yourgroup",
   "site.global.user_group":"yourgroup",
"site.hbase-site.hbase.superuser": "yourname"，
> 
如果你配置了HA，添加hdfs-site配置到这：
"site.hdfs-site.dfs.nameservices":"OCNoSQLBJ",
"site.hdfs-site.dfs.ha.namenodes.OCNoSQLBJ":"nn1,nn2",
"site.hdfs-site.dfs.namenode.rpc-address.OCNoSQLBJ.nn1":"ocean00:8020",
"site.hdfs-site.dfs.dfs.namenode.rpc-address.OCNoSQLBJ.nn2":"ocean00:8020",
"site.hdfs-site.dfs.namenode.shared.edits.dir":"qjournal://ocean00:8338;ocean01:8338/OCNoSQLBJ",
"site.hdfs-site.dfs.client.failover.proxy.provider.OCNoSQLBJ":"org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider",
"site.hdfs-site.dfs.ha.automatic-failover.enabled":"true",
3.3 启动slider实例
3.3.1启动:
python slider.py create slider4inst1 
--template appConfig.json --resources resources.json --image hdfs://yourNameNodeHost:port /slider/agent/slider-agent.tar.gz
3.3.2 slider 命令

- 添加regionserver
python slider.py flex slider4inst1 --component HBASE_REGIONSERVER 2
- 删除regionserver
python slider.py flex slider4inst1 --component HBASE_REGIONSERVER 1
- 	停止slider实例
python slider.py freeze slider4inst1
- 	再次启动
python slider.py thaw slider4inst1
- 	查看实例状态
python slider.py status slider4inst1
- 销毁实例
python slider.py destroy slider4inst1


 

