---
layout: post
title: slider client序列图
categories:
- 现象与使用
- slider 服务设计
tags:
- slider
- 服务
---

slider client序列图
=========


###python slider.py create...
Slider.main(args);
ServiceLauncher.serviceMain(extendedArgs);
serviceLauncher.launchServiceAndExit(服务发射台)
registerInterruptHandler
launchServiceRobustly
launchService
instantiateService
启停start runservice 结束。。
 
###具体的SlientClient服务实现
actionCreate:
actionBuild
buildInstanceDefinition
validateClusterName
verifyBindingsDefined
verifyNoLiveClusters
lookupZKQuorum
createClientProvider 
persist
 
     startCluster
      buildClusterDirPath
     loadInstanceDefinitionUnresolved
     launchApplication
          lookupZKQuorum
          verifyNoLiveClusters
          SliderAMClientProvider
          appOperations
          AbstractClientProvider
extractImagePath
provider.applicationTags
AppMasterLauncher
setMaxAppAttempts
purgeAppInstanceTempFiles
amLauncher.getLocalResources
//In this scenario, the jar file for the application master is part of the local resources


          getUsingMiniMRCluster
propagatePrincipals
sliderAM/provider.prepareAMAndConfigForLaunch
sliderAM/provider.preflightValidateClusterConfiguration 起飞预检

maybeAddImagePath
环境加载
rm address
addMandatoryConfOptionToCLI
amLauncher.addCommandLine()
sliderAM.prepareAMResourceRequirements
amLauncher.set*****
amLauncher.submitApplication();


     


