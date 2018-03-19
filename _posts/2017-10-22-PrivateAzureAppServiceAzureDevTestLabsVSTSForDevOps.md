---
layout: post
title: "Integrating VSTS for DevOps with privately hosted Azure App Service"
date: 2017-10-22
author: tarun
tags: ["DevOps", "Azure", "AzureAppService"]
categories:
- "DevOps"
image: "/images/screenshots/tarun/Sep17/AzureAppServiceVstsAzureDtl.png"
description: "Leverage Azure PaaS based Azure ASEv2 to privately host your web applications in isolation securely. This model of hosting gives you the true advantages of the cloud... In addition to this we saw how easy it is to leverage Azure Dev Test Labs to create a farm of private VSTS agents, this agent grid can be used to do seamless DevOps operations such as Continuous Deployment on your privately hosted Web Applications powered by ASE v2 Azure App Service."
permalink: /DevOps/PrivateAzureAseV2AppServiceVstsDeploymentUsingAzureDtl
published: true
keywords: "ASEv2, ASE, Azure, App Service, PaaS, App Service Plan, ASP, Private App Service, Isolated App Service, Private Subnet App Service, Azure Private Vnet App Service, Azure Private App Service, Azure App Service vNet Injection, Azure App Service Secure, Azure App Service block public access, Azure Dev Test Lab, Azure Dev Test Lab Grid, VSTS Agent Grid, VSTS Proxy Agent Build, VSTS Build Private Agent Proxy, VSTS Agent Enterprise Proxy, VSTS Agent Proxy configuration, Build Proxy, Release Proxy, VSTS Deploy Azure App Service ASEv2, ASEv2 Web Deployment, DevOps, Azure, Cloud, PaaS, Continuous Delivery"
---
Wondering how you can make the **Azure** App Service private to your network by totally blocking public discoverability? In this blogpost, I'll show you how and also show you how easy it is to use a grid of virtual machines in the cloud (in Azure Dev Test Lab), attached to the same private subnet to automate the deployment of your web application to the privately hosted Azure App Service. 
<!--more-->

It's encouraging to see that financial & trading organizations are starting to graduate from just talking about cloud to finally moving into the cloud. While CIO's love to brag about their move to the cloud, the more I speak to their teams the more I get the impression that they are limiting their use to just IaaS. Moving from virtual machines on-premise to IaaS in cloud is easier, but it doesn't necessarily give you the value you thought you would get from moving to the cloud. You still need to design the infrastructure, configure the operating system, worry about security patches and worst manage things like high availability and disaster recovery. It bedazzles me when enterprises choose to host their web applications on web servers on IaaS against hosting them on PaaS. More often than not, this is due to the limited knowledge of PaaS and a false perception that IaaS provides more isolation, control & security than PaaS.  

 > Did you know **Azure App Service "Environment" (ASE)** is a **PaaS** offering that gives you a completely **isolated private** space for **hosting** your apps in **Azure**? **ASE supports** isolated hosting of **Web Apps, Mobile Apps, API Apps and Functions**. Wait for it... Even supports use of **VSTS for DevOps!**  

![Surprise No Clue!]({{site.url}}/images/screenshots/tarun/Sep17/WhatSurpriseVstsDevOpsAzurePaaSDtl.gif)

# ASE v2 - App Service Environment 
In case you were living on another planet and missed the announcement... Azure ASE v2 GA'ed on 27th July 2017, this is an upgrade to the existing ASEv1 that has been an Azure offering for a while now. App Service Environment (ASE) is powerful, it gives *network isolation* and improved scale capabilities. It is essentially a deployment of the Azure App Service into a subnet of a _customer Azure Virtual Network (VNet)_. ASEv2 provides true multi tenancy like you've been used to with PaaS services in Azure. 

<p align="center">
<img src="https://www.datapipe.com/images/uploads/img/gifs/header-security-managment.gif" alt="Related image"/>
</p>

So, what you waiting for...? Grab your self the [ASE v2 ARM template](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-ilb-create) from the Azure quick start template repository on GitHub and provision yourself an instance of the Azure App Service Environment connected to your subnet in your very VNet. Here is the app service environment I have provisioned... 

![Azure App Service ASEv2]({{site.url}}/images/screenshots/tarun/Sep17/image-9e0b26c4-7387-4a41-a3b1-f33b00865fc2.png)

There is an internal load balancer for the app service environment...

![Azure APP Service ASEv2 ILB]({{site.url}}/images/screenshots/tarun/Sep17/image-c416655b-ae93-4cfd-92ea-84da0c35065c.png)

> You have to be careful how to setup the NSG rules, any rules that block communication between the ASE will stop it from working. Microsoft has good documentation on for [NSG set up for ASEv2](https://docs.microsoft.com/en-us/azure/app-service/app-service-environment/network-info#network-security-groups)... 

Now add an App Service Plan... ASP's are a great way to split the hosting of the app's on the ASE, as you can see below I have created three ASP's... 

![Azure App Service Plan for ASE v2]({{site.url}}/images/screenshots/tarun/Sep17/image-97780f0a-7537-4e6f-a9f0-d03ea439e5c7.png)

I've also tweaked the logic in the ASE to add an additional Front end node for every 5 ASP instances added to the ASE...

![ASE v2 Front End Node]({{site.url}}/images/screenshots/tarun/Sep17/image-74968cc5-c5b5-433f-b49a-fe6be908e33c.png)

Alright, time to create the first web app... As you can see the web app inherits the ILB and subdomain specified as well as the network setup specified in the ASE...

![Azure App Service on ASEv2 ASP]({{site.url}}/images/screenshots/tarun/Sep17/image-9fedf330-1be5-475c-98d6-2528dd45c96d.png)

Alright with the setup of the ASEv2, App Service Plan and Web App out of the way, let's focus on getting DevOps going with VSTS... 

# How do I get DevOps going with Azure ASEv2 App Service?
---
Now that you have a web app, you would want to create build and release pipelines to get a continuous delivery workflow for your application. VSTS provides hosted agents, but since we are in a private Subnet these hosted Agents will not have connectivity into the web app. In this case you'll need to set up your own privately hosted agent. You can set up a virtual machine in the same subnet as the ASEv2... You can read more about how to set up a [private VSTS agent](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-windows). 

### VSTS Private Agent behind Enterprise Proxy 
The VSTS Agent supports working with an Enterprise proxy natively... Key points to remember...

+ Create .proxy file with proxy url under agent root directory.
+ If using an authenticated proxy, set authenticate proxy credential through environment variable
VSTS_HTTP_PROXY_USERNAME and VSTS_HTTP_PROXY_PASSWORD 

More details on how to set up the VSTS Private Agent with [proxy here](https://github.com/Microsoft/vsts-agent/blob/master/docs/start/proxyconfig.md) 

![VSTS Private Agent Proxy Config Test Validate]({{site.url}}/images/screenshots/tarun/Sep17/VstsAgentBehindProxy.JPG)


You know me, i don't do small... So instead of provisioning one agent in the same subnet, I have provisioned the Azure Dev Test Lab in the same virtual network as the ASEv2 instance and added the ASEv2 subbnet to the AzureDevTestLab. This will allow me to create multiple agents and benefit from the capabilities offered by Dev Test Labs, learn more about [Azure Dev Test Labs](http://www.visualstudiogeeks.com/blog/DevOps/Use-VSTS-ReleaseManagement-to-Deploy-and-Test-in-AzureDevTestLabs). The diagram below reflects the network connectivity between my ASEv2 instance and the Azure Dev Test Labs. 

![Private VSTS Agent Farm with Azure DTL]({{site.url}}/images/screenshots/tarun/Sep17//image-8d4b634e-2005-49a1-a1c0-d41a44010759.png)

# Deployment Pipeline in VSTS for Azure ASEv2 App Service 
---
From the build hub, create a new build definition, choose the out of box template `Azure Web App`. This will load all the relevant tasks in the build pipeline that will let you build, unit test and deploy a web application to an Azure App Service. 

![ASEv2 Azure App Service Deployment with VSTS]({{site.url}}/images/screenshots/tarun/Sep17/BuildTemplateForAzureDeploy.JPG)

The defaults in the template are enough to get you a web deploy package that is what you need to publish your web application to Azure Web App. 

> However, the default template points to the HOSTED Visual Studio 2017 agent. While this agent will be able to create the web deploy package, it won't be able to deploy to your ASE v2 Azure App Service as the App Service won't be discoverable to the agent. 

In order to get the web deploy package out to the Azure App Service, you'll need to change the `Agent Queue` to the agents you've set up in Azure Dev Test Lab (which are in the same subnet as the Azure ASE v2 instance). 

![VSTS ASEv2 App Service Private Agent Queue]({{site.url}}/images/screenshots/tarun/Sep17/VstsPrivateBuildQueueForAsev2Deploy.JPG)
 
And that's really all you need to get the deployment going. Queue a new build job and see this in action... 

![Azure App Service VSTS Deployment Pipeline with VSTS]({{site.url}}/images/screenshots/tarun/Sep17/DeployAzureAppService.JPG)


> Pro Tip - If you don't have a certificate set up for your Azure App Service yet, the deployment is likely to fail with the error message ` ##[error]Error Code: ERROR_CERTIFICATE_VALIDATION_FAILED`. Full error message details below... Use `-allowUntrusted` flag in the deploy task to force the deploy ignoring the certificate trust.  

``` cmd 
##[error]Failed to deploy web package to App Service.
2017-10-06T14:32:13.8838607Z ##[error]Error Code: ERROR_CERTIFICATE_VALIDATION_FAILED
More Information: Connected to the remote computer ("xxx-01.scm.azsu-ase-devtest-l-shared-xxx-001.azure.uk.xxx.com") using the specified process ("Web Management Service"), but could not verify the server’s certificate. If you trust the server, connect again and allow untrusted certificates.  Learn more at: http://go.microsoft.com/fwlink/?LinkId=221672#ERROR_CERTIFICATE_VALIDATION_FAILED.
Error: The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.
Error: The remote certificate is invalid according to the validation procedure.
Error count: 1. 
```

![VSTS Azure App Service Error Certificate AllowUntrusted]({{site.url}}/images/screenshots/tarun/Sep17/VstsAzureAppDeployWithOutCertAllowUntrusted.JPG)

Wondering how you transfer the configuration files or tokenizing them to make them environment agnostic. Check out the post here which shows you the [most efficient way to tokenize and parametrize configuration files using VSTS](http://www.visualstudiogeeks.com/DevOps/TransformWebConfigFileWithVSTSWithoutTokenizationOrParametersXml).  

# Summary 
In this blogpost we saw how easy it is to leverage Azure PaaS to privately host your web applications in isolation securely. This model of hosting gives you the true advantages of the cloud... In addition to this we saw how easy it is to leverage Azure Dev Test Labs to create a farm of private VSTS agents, this agent grid can be used to do seamless DevOps operations such as Continuous Deployment on your privately hosted Web Applications powered by ASE v2 Azure App Service. 

Hope you found this useful... any questions feel free to reach out on [@arora_tarun](https://twitter.com/arora_tarun)

Tarun 



 








