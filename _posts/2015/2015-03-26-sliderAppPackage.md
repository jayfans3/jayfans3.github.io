---
layout: post
title: slider打组件包和组件安装使用命令
categories:
- 逻辑与现象
- slider 使用
tags:
- slider
- 服务
---


##slider打组件包和组件安装

###最新使用：

	mvn clean package -Phbase-app-package -Dpkg.version=0.98.4-hadoop2 -Dpkg.name=hbase-0.98.4-hadoop2-bin.tar.gz -Dpkg.src=/home/ocean/app/oceanslider
	
	
	python slider.py install-package --name HBASE --package /mnt/hgfs/apache_git_src/incubator-slider/app-packages/hbase/target/slider-hbase-app-package-0.62.0-SNAPSHOT.zip --replacepkg --fs ocean00:9000
	
	
	mvn clean package -Phbase-app-package -Dpkg.version=0.98.4-hadoop2 -Dpkg.name=hbase-0.98.4-hadoop2-bin.tar.gz -Dpkg.src=/home/ocean/app
	mvn clean package -Phbase-app-package -Dpkg.version=0.98.6.1-och4.0.0 -Dpkg.name=hbase-0.98.6.1-och4.0.0.tar.gz -Dpkg.src=/home/ocean/app
	
	
	python slider.py install-package --name HBASE --package /home/ocean/app/build/incubator-slider-release-0.60.0/app-packages/hbase/target/slider-hbase-app-package-0.60.0-incubating.zip --replacepkg


官方
-----------------


Use the following command to install HBase tarball locally:
  mvn install:install-file -Dfile=<path-to-tarball> -DgroupId=org.apache.hbase -DartifactId=hbase -Dversion=0.98.3-hadoop2 -Dclassifier=bin -Dpackaging=tar.gz

You may need to copy the hbase tarball to the following location if the above step doesn't publish the tarball:
~/.m2/repository/org/apache/hbase/hbase/0.98.3-hadoop2/

After HBase tarball is published locally in maven repository, you can use the following command:
  mvn clean package -DskipTests -Phbase-app-package

App package can be found in
  app-packages/hbase/target/apache-slider-hbase-${hbase.version}-app-package-${slider.version}.zip

Verify the content using
  zip -Tv apache-slider-hbase-*.zip


eg:
D:\>mvn install:install-file -Dfile=hbase-0.98.3-hadoop2-bin.tar.gz -DgroupId=org.apache.hbase -DartifactId=hbase -Dversion=0.98.3-hadoop2 -Dclassifier=bin -Dpa
ckaging=tar.gz

D:\apache-slider-0.40.0-incubating-source-release\apache-slider-0.40>mvn clean package -DskipTests -Phbase-app-package

Desired
****** OPTION - II (manual) **
The Slider App Package for HBase can also be created manually.

Download the tarball for HBase:
  e.g. path to tarball ~/Downloads/hbase-0.98.3-hadoop2-bin.tar.gz

Copy the hbase tarball to package/files
  cp ~/Downloads/hbase-0.98.3-hadoop2-bin.tar.gz package/files

Edit appConfig.json/metainfo.xml
  Replace 4 occurrences of "${hbase.version}" with the hbase version values such as "0.98.3-hadoop2"
  Replace 1 occurrence of "${app.package.name}" with the desired app package name, e.g. "hbase-v098"

Create a zip package at the root of the package (<slider enlistment>/app-packages/hbase/)
  zip -r hbase-v098.zip .

Verify the content using
  zip -Tv hbase-v098.zip






	* Application Needs What it takes to be deployable by Slider.
	* Slider AppPackage Overview of how to create an Slider AppPackage.
	* Specifications for AppPackage Describes the structure of an AppPackage
	* Specifications for Application Definition How to write metainfo.xml?
	* Specification of Resources How to write a resource spec for an app?
	* Specifications InstanceConfiguration How to write a template config for an app?
	* Guidelines for Clients and Client Applications
	* Specifications for Configuration Default application configuration
	* Specifying Exports How to specify exports for an application?
	* Documentation for "General Developer Guidelines"
	* Configuring the Slider Chaos Monkey

