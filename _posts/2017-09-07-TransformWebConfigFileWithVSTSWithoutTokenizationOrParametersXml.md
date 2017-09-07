---
layout: post
title: "How to transform Web.Config file 'Properly' with VSTS!"
date: 2017-08-26
author: tarun
tags: ["DevOps", "BuildPipeline"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/Sep17/TransformConfigNativelyInVsts.gif"
description: "VSTS now natively supports config transformations, if you are still using config per environment or parameters.xml then you are doing it wrong."
permalink: /DevOps/TransformWebConfigFileWithVSTSWithoutTokenizationOrParametersXml
published: true
keywords: "DevOps, VSTS, WebConfigTransform, WebConfigTokenization, ConfigTransform, Config Transform per environment, Config Transform VSTS, Release Pipeline config Transform, Config Transform Web Deploy, Config Transform without tokenizer, Config Tranform dynamically with VSTS, Release Pipeline, Build Pipeline, Azure App Service Config Tranform, Config Transform Continuous Delivery"
---
Web development and the frameworks that go with it have evolved over the years, but when it comes to transforming config file people still use the old ways! If you are still using one of these approaches, you need to change...  
- One configuration file per environment
- Tokenization of configuration files using parameters.xml and then using a tokenizer to replace values during deployment
<!--more-->

> # **VSTS now natively supports config transforms!**  

<p align="center">
<img src="https://media.tenor.com/images/e986f3cda38e718a181ce57cfad77fe4/tenor.gif" alt="Related image"/>
</p>

Visual Studio Team Services offers the `Azure App Service Deploy` task, version 3 of this task now natively supports config transformation. Let's see this in action... 
![image.png](/images/screenshots/tarun/Sep17/image-4c46e477-bd71-4f35-82b1-1a5ccc4d3cd1.png)

Scroll down to the File Transforms & Variables Substitution Options section and check the option `XML variable substitution`
![image.png](/images/screenshots/tarun/Sep17/image-7d1c259c-67ee-4592-916b-4b822be87d7a.png)

Navigate to the variables section in the release definition and create a key to replace the connection string with a value for a specific environment. 
![image.png](/images/screenshots/tarun/Sep17/image-8883ea42-a054-4b02-945c-809e81d0c38b.png)

> You have the option to encryt the value and also specify the scope of the variable to an environment or the entire definition. 

Now fire the release and let this task execute, look at the logs, you'll see the transformation has been completed for you...

![image.png](/images/screenshots/tarun/Sep17/image-4f30412e-4b1e-4944-8b20-64abde03aeb1.png)

Let's look at the value in the config file...

![image.png](/images/screenshots/tarun/Sep17/image-6b881282-937f-49d5-a9aa-50fc81e454da.png)

No more going back to the old ways of transforming webconfig files... VSTS has you covered. #DevOpsOn

Tarun 