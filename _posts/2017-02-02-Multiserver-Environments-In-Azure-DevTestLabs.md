---
layout: post
title: "Using VSTS to provision multi server environments in Azure Dev Test Labs"
date: 2017-01-04
author: utkarsh
tags: ["Extensions"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
description: "Azure Dev Test Labs support single server environments, previously we've seen how to use the VSTS Azure Dev Test Labs extension from the Visual Studio Marketplace to automate the provisioing of virtaual machines in the Azure Dev Test Labs. In this blog post we'll see how to deploy multi server environments in Azure Dev Test Labs using Arm deployment task in VSTS."
permalink: /DevOps/multiserver-environments-in-azure-devtestlabs
keywords: "Extensions"
---

At launch Azure Dev Test Labs only supported single server environments. While this was great for development and standalone testing, this limited the use of Dev Test Labs for any meaningful testing. If you are truely following DevOps you would like your automated environment provisioing to resemble production like environments throughout from development all the way to production. After all it's all about pushing left and testing your automation from the outset ;) 
<!--more-->
# Introduction
As one of the most voted features, it came as no suprise when @ Connect() 2016 the Microsoft Azure team announced the availability of Bring Your Own Template (BYOT) support in Azure Dev Test Labs. BYOT basically means support for custom ARM templates, an ARM template could be used to create a single server or complex multi server environment that could comprise of both IaaS and PaaS resources. [Xiaoying Guo](https://social.msdn.microsoft.com/profile/Xiaoying+Guo) a program manager in the Azure Dev Test Labs team has a great blogpost on [How to set up custom multi server ARM based environments in Azure Dev Test Labs?](https://blogs.msdn.microsoft.com/devtestlab/2016/11/16/connect-2016-news-for-azure-devtest-labs-azure-resource-manager-template-based-environments-vm-auto-shutdown-and-more/). 

> In this blogpost I am going to take this even a step further and show you how to leverage Visual Studio Team Services or Team Foundation Server Release Pipelines to orchestrate the deployment of this complex environment to DevTestLabs using the Azure Resource Group Deployment task.



