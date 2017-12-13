---
layout: post
title: "Automate DevSecOps with CI CD using VSTS..."
date: 2017-12-13
author: tarun
tags: ["DevOps", "DevSecOps", "VSTS", "Azure"]
categories:
- "DevOps"
- "DevSecOps"
img: "/images/screenshots/tarun/Dec17/SecurityIsABottleNeck.jpg"
description: "Security your cloud infrastructure doesn't have to be a daunting task. Luckily with VSTS and now with Jenkins, it's possible to leverage the CICD pipeline and the open source AzSDK extension to inspect Azure PaaS and Azure IaaS for best practice compliance. It gives you the ability to move from DevOps to DevSecOps and empower your development teams to release features securely at speed."
permalink: /DevOps/AutomateDevSecOpsWithCICDPipelineUsingVSTSAzSDKExtension
published: true
keywords: "DevSecOps, SecurityOps, SecurityPipeline, SecurityCompliance, Security Ops, Ruggid DevOps, Azure IaaS Security, Azure PaaS Security, Azure IaC Security, Azure Security, Azure Infrastructure Security, Azure Security Automation, Cloud Security Automation, Azure Security CI CD, Security CI CD, VSTS CI CD Security, Security CI CD, Security Automation"
---
With the adoption of infrastructure as code and development teams taking on more of the `Ops` activities, organizations using Azure for enterprise hosting are practically at risk of breach with every new release they roll out to these environments... In this blogpost I'll walk you through `AzSDK Security Verification Tests` that can be setup in the CICD pipeline using VSTS to automate the inspection of your infrastructure in Azure and block releases that can potentially compromise your infrastructure.      
<!--more-->

The DevOps way of working coupled with the use of cloud technologies has practically removed the biggest barriers from the software delivery life-cycle... Development teams don't care about security as much as they should. It's hard to blame the development teams though, the tools used by security are not easy to understand… The reports are long and hard to interpret... **The approach of compliance driven security doesn't practically fit into the fast pace of software delivery we are used to today.** 

![Security is a bottleneck in DevOps](/images/screenshots/tarun/Dec17/SecurityIsABottlneckInDevOps.jpg)

> **DevSecOps** helps bring a fresh perspective by introducing a culture of making __everyone accountable__ for security, using automation to move the process of inspection _left_ and overall looking at security with a 360 lense.

# Size of the problem...
I have a resource group in Azure dedicated to the application in question, only selective people have access to this resource group. I am using Web App's, Azure Functions, Blob Storage, Redis Cache, App Insights, Service Bus, SQL warehouse among some other Azure resource types. *This is just one of the hundreds of resource groups I have in the tens of Azure subscriptions used by this organization. The ratio of security consultants to the number of releases makes it practically impossible for security to inspect the changes to guarantee compliance to best practices.* 

# Azure + VSTS + CICD + AzSDK = DevSecOps 
Start by installing the AzSDK extension from the [vsts marketplace](https://marketplace.visualstudio.com/items?itemName=azsdktm.AzSDK-task)

![AzSDK VSTS Marketplace](/images/screenshots/tarun/Dec17/AzSdk-Marketplace.jpg)

There is some amazing documentation on the [azSdk GitHub repo](https://github.com/azsdk/azsdk-docs/blob/master/03-Security-In-CICD/Readme.md#security-verification-tests-svts-in-VSTS-pipeline)

### I'll show you the extension in action... 

+ Create a new release pipeline is VSTS, add an Azure environment you want to run the extension against... 

![AzSDK VSTS - New Release Pipeline](/images/screenshots/tarun/Dec17/AzSdk-VstsReleasePipelineNewEnv.jpg)

+ Add the azSdk task and configure the VSTS SPN to the Azure subscription, specify the name of the Azure resource group you wish to inspect as well as the Azure subscription id the resource group resides in. 

The task also gives you the option to log the results in Azure OMS, you can find out more details along with other advanced options [here](https://github.com/azsdk/azsdk-docs/blob/master/03-Security-In-CICD/Readme.md#enable-azsdk-extension-for-your-vsts)... 

![AzSDK VSTS - AzSDK Task Configuration Example](/images/screenshots/tarun/Dec17/AzSdk-VstsRmTaskConfigurationExample.jpg)

+ Run the release pipeline and wait for the release to complete... The default security policies are evaluated, you have the option of customizing or creating your subset... If the setup isn't compliant the release pipeline will fail by default... 

![AzSDK VSTS - AzSDK Task Configuration Example](/images/screenshots/tarun/Dec17/AzSdk-NonCompliantSetup.jpg)

> Looking at the detailed logs blew my mind! 

![AzSDK VSTS - AzSDK Analysis Results](/images/screenshots/tarun/Dec17/AzSdk-ResultsCsv.jpg)

The logs give you the full picture! The extension gives you a criticality rating and also tells you if the issue can be auto fixed, a description and recommendation for the specific line item as well... 

![AzSDK VSTS - AzSDK Analysis Results](/images/screenshots/tarun/Dec17/azSdk-FullAnalysis.jpg)

# Summary 
Pushing Security left does not mean using the old ways of security compliance checks earlier in the life-cycle, instead it is an opportunity to leverage the DevOps mindset to innovate and automate the security inspection to allow development teams to release features with confidence.  

Tarun   



