---
layout: post
title: "HelloWorld Chef on Windows, Knife, Cookbook, IIS, Windows Azure & everything in between..."
date: 2017-07-03
author: tarun
tags: ["DevOps", "Chef"]
categories:
- "DevOps"
image: "/images/screenshots/tarun/chefonwindows_azure.jpg"
description: "Zero to Hero with Chef! In this blogpost we'll see how to set up Chef from scratch on Windows. We'll then work our way up to authoring and deploying our first cookbook using Knife to manage the desired state configuration (DSC) of a windows Azure VM. Configuration Managemnt is one of the key pillars of DevOps... In this blogpost we'll see how to get this going with Chef... "
permalink: /DevOps/SetupChefOnWindowsAuthorDeployCookbookUsingKnifeAzureVm
published: true
keywords: "DevOps, InfastructureAsCode, IaC, ConfigurationManagement, Configuration Management, Desired State Management, DSC, Chef, Chef Tools, Chef in Azure, Chef Azure VM Extension, Chef Knife client, Knife, Windows Azure VM, Windows, Cookbook, IIS Cookbook"
---
Configuration Management is one of the key pillars of DevOps. Configuration Management helps address snowflake servers and configuration drift. Microsoft has it's own offering for configuration management in azure with hosted DSC server through Azure Automation. Microsoft has an equally compelling story with the open source tools such as Chef and Puppet. While Chef is truly amazing in it's capabilities, getting it going on a Windows machine can be challenging due to the various moving parts in it's configuration... In this blogpost I'll show you what it takes to get started with Chef by setting up Chef server on Windows, creating your first cookbook, uploading it to Chef using Knife and then using this runbook to manage a Windows host in Azure using the Chef Azure extension... Quite a mouthfull... Grab a coffee, let's go through this together... 
<!--more-->

Let's start off by looking at the conceptual architecture for Chef... Chef has three main architectural components: Chef Server, Chef Client (node), and Chef Workstation. 

+ Chef Server: Setting up the Chef Server on Windows
+ Administrator Workstation: Set up a development workstation with Chef Tools to author the cookbooks and uploading them on to chef server 
+ Microsoft Azure: Provision a new Azure VM with Chef Agent that integrates with Chef server and registers itself to deploy the cookbook registered under its profile in Chef

![Chef Server Architecture]({{site.url}}/images/screenshots/tarun/ChefServerArchitectrueDiagram.png)

The Chef Server is the management point and there are two options for the Chef Server: a hosted solution or an on-premises solution. In this case, we'll be using the hosted Chef Server, but you'll still need a Chef Administrator Workstation which in my opinion requires almost the same level of configuration if not more. The Chef Workstation is the admin workstation where we create our policies and execute our management commands. We run the knife command from the Chef Workstation to manage our infrastructure.There is also the concept of “Cookbooks” and “Recipes”. These are effectively the policies we define and apply to our servers.

I've created a video tutorial to demonstrate how to accomplish this setup... Follow along... If you have any questions please leave me a comment and i'll try my best to helpout... 

{% include youtubeplayer.html id='EctKInvVfhU' %}

``` PowerShell













```

If you are looking for more information on how to go from continuous integration to continuous deployment, check out my other course on DevOps that covers exactly that... You can check out the course here - [DevOpsCiCdCourse](http://bit.ly/DevOpsCiCdInfo)

Tarun 