---
layout: post
title: "Replace appsetting tokens in config files with Build & Release Management in VSTS (TFS)"
date: 2016-11-05
author: utkarsh 
tags: ["DevOps", "ReleaseManagement"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
description: "Update app settings replace tokens in config files with build and release management in VSTS and TFS"
permalink:  /DevOps/replace-appsettings-token-configfiles-build-release-tfs-vsts
keywords: "DevOps"
---
Whether you are developing for web or windows. Replacing the configuration variables persisted in the web/app.config is always a pain during the build and deployment process using TFS. In this walkthrough we'll learn how to use the "ReplaceTokens" task during build and deployment to update the configuration easily. 

<!--more--> 
# Pre-Requisites 
This walk through assumes you already have a web based project set up in Team Foundation Server or Visual Studio Team Services. 

# Walkthrough 
1. Open Team Foundation Server and navigate to the marketplace by clicking on the 'shopping bag' icon from the top right corner in the navigation bar.
   ![NavigateToMarketplace](/images/screenshots/tarun/ReplaceTokens/Nav2MrktplaceVstsTfs.png)

2.	In the extension marketplace search for replace tokens, download and add this extension to TFS. You can find the replace token extension by browisng to [ReplaceToken Task in Marketplace](https://marketplace.visualstudio.com/items?itemName=qetza.replacetokens&targetId=4ace6815-ca0a-4926-92f1-862643b5c950). By default collection administrators have permissions to approve extensions added to a team foundation server. 
   ![ReplaceTokenExtension](/images/screenshots/tarun/ReplaceTokens/ReplaceTokenExtension.png)

3.	Navigate to the build hub and create a new build definition. In the example build pipeline below, I have used the visual studio build template that comprises of build, test and package.
   ![DefaultBuildPipeline](/images/screenshots/tarun/ReplaceTokens/DefaultBuildPipeline.png)
 
4.	Click the add build step link to add the replace tokens task to the build pipeline. The target files field accepts a wild card keyword, the default will look for all config files in your repository. As highlighted in the advanced section, you can change the prefix and suffix value based on the convention used in your project. 
   ![ReplaceTokenTask](/images/screenshots/tarun/ReplaceTokens/ReplaceTokenTask.png)
 
5.	Let's look at the config file I am planning to update with this replace tokens task. As you can see in the below screen shot, I am looking to replace the appSetting key "Variable1" the current value of variable1 key is "#{Variable1}#". 
   ![AppConfigVariable](/images/screenshots/tarun/ReplaceTokens/AppConfigVariable.png)
 
6.	In order to replace the value of variable1 in the web.config, I am going to configure the "replace token" configuration.
   ![BuildTokenVariableReplace](/images/screenshots/tarun/ReplaceTokens/BuildTokenVariableReplace.png)
 
7.	Now navigate to the variables tab in your build definition and add a new variable that matches the name of the value field in the key under appSettings file that you are planning to replace. 
   ![ReplaceTokenVariableExample](/images/screenshots/tarun/ReplaceTokens/ReplaceTokenVariableEx.png)
 
8.	Save the build definition and queue a build. The replace token task should execute and update the logs. See screen shot below. 
   ![BuildOutputReplaceToken](/images/screenshots/tarun/ReplaceTokens/BuildOutputReplaceToken.png)
 
9.	To ensure that the configuration file has been successfully updated, navigate to the folder agent working folder in the server D:\DCDWHTFSB01_A1\_work\7\s\Main\Source\Edft.Web.UI and open the web.config file. As indicated in the screen shot below the value of the key "Variable1" has successfully been updated by the variable specified in the build definition. 
   ![ReplaceTokenLogFileOutput](/images/screenshots/tarun/ReplaceTokens/ReplaceTokenLogFile.png)
  
10.	This task can be used during build or release. The results will be the same. 
 
> The great benefit of this task is that you don't need to separately manage the variable values in config files or parameter files, instead you can simply update the values directly from the build definition by maintaining the master copy of the variables in the build definition directly. What's even cooler is that the build definition allows you to store variables encrypted as well. This is ideal for passwords and sensitive values that now don't need to reside in configuration files and can instead directly be applied from build definitions directly. 



