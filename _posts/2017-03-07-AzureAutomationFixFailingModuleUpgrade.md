---
layout: post
title: "Azure Automation - Fix Failing Module Upgrade in Azure Portal"
date: 2017-03-07
author: tarun
tags: ["DevOps", "Azure", "AzureAutomation"]
categories:
- "DevOps"
image: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
description: "Azure Automation Azure Modules Upgrade fails PowerShell run book fails"
permalink: /DevOps/AzureAutomationFixFailingModuleUpgradeInAzurePortal
keywords: "DevOps, Azure, AzurePortal, AzureAutomation, AzureRM Modules, PowerShell Run Books"
---

If your Run Books in Azure Automation rely on any of the new versions of the AzureRM PowerShell module, then they may start to fail! That's because all the modules that get pre-added in Azure Automation point to an older version of AzureRM modules... Follow the steps in this post to fix failing module upgrade...
<!--more-->

The screen shot below shows you the Module pane in AzureAutomation, you could click the update azure modules button to auto upgrade the modules. This functionality allows you to force an upgrade of the modules, this would update the modules to a version that are compatible with each other. However, this functionality seems to be broken...

![AzureAutomation-UpdateAzureModules]({{site.url}}/images/screenshots/tarun/AzureAutomation/AzureAutomationUpdateAzureModules.PNG)

### Error Message 

![AzureAutomation-UpdateModule-ErrorMessage]({{site.url}}/images/screenshots/tarun/AzureAutomation/AzureAutomationModuleUpgradeFailureErrorMessage.PNG)

### Fix 

After hours of troubleshooting, it came to light that the AzureRm.Profile and AzureRM.Automation should point to the same version, if they are on separate versions then Azure Automation __silently starts__ to fail the module upgrade process. This can however be fixed by deleting the AzureRM.Profile module from the modules list and then forcing the upgrade of the modules.


![AzureAutomation-AzureRMProfileModuleDelete]({{site.url}}/images/screenshots/tarun/AzureAutomation/AzureRMProfileModuleDelete.PNG)

This will download the AzureRM.Profile module to the correct version that matches the version of AzureRM.Automation and then continuous to perform the upgrade of all the modules successfully. 

> The fix is simple, but just putting this out there because it took hours to figure the root cause...

Tarun
