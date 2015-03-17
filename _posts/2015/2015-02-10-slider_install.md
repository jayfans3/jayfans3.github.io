---
layout: post
title: slider-0.70安装
categories:
- 使用
tags:
- slider
- 安装
---

###启动minicluster集群,zk,rm,nm nn,dn,ssh
./hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-tests.jar minicluster  -rmport 8032 -jhsport 9080 -nnport 9000 -namenode hdfs://ocean00:9000 -Dyarn.nodemanager.delete.debug-delay-sec=256 -Dyarn.nodemanager.delete.debug-delay-sec=3600
### slider修改env.sh java_home hadoop_conf_dir
###创建hdfs目录/user/{user}
### 制作hbasezip
mvn clean package -Phbase-app-package -Dpkg.version=0.98.4-hadoop2 -Dpkg.name=hbase-0.98.4-hadoop2-bin.tar.gz -Dpkg.src=/home/ocean/app/oceanslider
(注意-bin去掉)
###上传zip
python slider.py install-package --name HBASE --package /mnt/hgfs/apache_git_src/incubator-slider/app-packages/hbase/target/slider-hbase-app-package-0.62.0-SNAPSHOT.zip --replacepkg --fs ocean00:9000
### 配置appconfig resource 
### 启动slider
python slider.py create oceantest --template appConfig.json --resources resources.json --manager ocean00:8032 --fs ocean00:9000
以下命令测试使用：
> 
python slider.py stop oceantest --manager ocean00:8032 --fs ocean00:9000
python slider.py registry --list  --manager ocean00:8032 --fs ocean00:9000
python slider.py registry --listconf --name hbase  --manager ocean00:8032 --fs ocean00:9000
python slider.py status  --manager ocean00:8032 --fs ocean00:9000
python slider.py registry --getconf quicklinks --name hbase  --manager ocean00:8032 --fs ocean00:9000
python slider.py flex hbase --component HBASE_MASTER 2  --manager ocean00:8032 --fs ocean00:9000
python slider.py flex hbase --component HBASE_REGIONSERVER 8  --manager ocean00:8032 --fs ocean00:9000
python slider.py registry --listconf --nameoceantes1t--manager ocean00:8032 --fs ocean00:9000
python slider.py registry --getconf quicklinks --name oceantes1t--manager ocean00:8032 --fs ocean00:9000
python slider.py registry --getconf core-site --name oceantest --manager ocean00:8032 --fs ocean00:9000 --out core-site.xml
python slider.py registry --getconf hbase-site --name oceantest --manager ocean00:8032 --fs ocean00:9000 -out hbase-site.xml
python slider.py registry --getconf hdfs-site --name oceantest --manager ocean00:8032 --fs ocean00:9000 --out hdfs-site.xml

