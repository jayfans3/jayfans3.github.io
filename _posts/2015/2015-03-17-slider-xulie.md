---
layout: post
title: slider client序列
categories:
- 逻辑与现象
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


     
###service端的SliderAmMaster实现

createAndRunCluster:


	// initAndAddService(providerService);
	//sliderAMProvider = new SliderAMProviderService();
	//yarnRPC = YarnRPC.create(serviceConf);
	// asyncRMClient = AMRMClientAsync.createAMRMClientAsync(heartbeatInterval, this);
	//nmClientAsync = new NMClientAsyncImpl("nmclient", this);
	//startSliderRPCServer();
	//startRegistrationService();
	// startAgentWebApp(appInformation, serviceConf);
	//webApp = new SliderAMWebApp(registry);
	//new WebAppService<SliderAMWebApp>("slider", webApp);
	//appState.buildInstance(instanceDefinition,
	          serviceConf,
	          providerConf,
	          providerRoles,
	          fs.getFileSystem(),
	          historyDir,
	          liveContainers,
	          appInformation,
	          new SimpleReleaseSelector());
	//launchService = new RoleLaunchService(actionQueues,
	                                          providerService,
	                                          fs,
	                                          new Path(getGeneratedConfDir()),
	                                          envVars,
	                                          launcherTmpDirPath);
	// sliderAMProvider.start();
	    // launch the real provider; this is expected to trigger a callback that
	    // starts the node review process
	//launchProviderService(instanceDefinition, confDir);providerService start
	//重用锁



