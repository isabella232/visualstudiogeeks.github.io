---
layout: post
title: "Using VSTS to provision multi server environments in Azure Dev Test Labs"
date: 2017-01-04
author: tarun
tags: ["DevOps", "Azure", "AzureDevTestLabs"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
description: "Azure Dev Test Labs support single server environments, previously we've seen how to use the VSTS Azure Dev Test Labs extension from the Visual Studio Marketplace to automate the provisioing of virtaual machines in the Azure Dev Test Labs. In this blog post we'll see how to deploy multi server environments in Azure Dev Test Labs using Arm deployment task in VSTS."
permalink: /DevOps/multiserver-environments-in-azure-devtestlabs
keywords: "DevOps, Azure, AzureDevTestLabs, TeamBuild, Continuos Deployments, Release Management"
---

If your Run Books in Azure Automation rely on any of the new versions of the AzureRM PowerShell module, then they may start to fail! That's because all the modules that get pre-added in Azure Automation point to an older version of AzureRM modules... Follow the steps in this post to fix failing module upgrade...
<!--more-->

The screen shot below shows you the Module pane in AzureAutomation, you could click the update azure modules button to auto upgrade the modules. This functionality allows you to force an upgrade of the modules, this would update the modules to a version that are compatible with each other. However, this functionality seems to be broken...

### Error Message 

![AzureAutomation-UpdateModule-ErrorMessage](/images/screenshots/tarun/AzureAutomation/AzureAutomationModuleUpgradeFailureErrorMessage.PNG)

