---
layout: post
title: "How to move custom images (vhd) between Azure DevTest Labs"
date: 2016-05-10 07:25:00 
author: Tarun Arora 
tags: ["DevOps", "Azure", "AzureDevTestLabs"]
categories:
- blog                #important: leave this here
- "DevOps"
permalink:  /blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs
description: "DevOps Azure DevTestLab InfrastructureAsCode VSTS AzureResourceManager InfrastructureAutomation VHD CustomImages"
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /images/screenshots/thumbs/
---
Like me if you are using multiple Dev Test Labs in Azure for multiple teams, you would like to share custom images created by one group to the other if it helps them accelerate. After all sharing is caring :) In this blog post we'll learn how to move custom images aka VHD's between Azure Dev Test Labs. It would be great if you could leave a comment if you know a better way of achieving this... I would also be interested to know what other assets are you sharing between Azure Dev Test labs...
<!--more--> 

_You might find this interesting..._ [Deploying a new VM in an existing Azure DevTest Lab using VSTS](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS)

# Scenario
The Azure Dev Test Labs reside in a resource group that contains the lab itself, network, optionally a network security group and two storage accounts. 
![AzureDevTestLab](/images/screenshots/tarun/AzureDTL/AzureDtl_ResourceGroupDiagram.png)

One of these two storage accounts is used to store the custom images generated in the dev test lab. If we navigate into the storage account and select blob storage, we'll be able to locate a container that's storing the generated vhds. This blob container stores the custom images generated in the dev test lab. 

![AzureDevTestLab](/images/screenshots/tarun/AzureDTL/AzureDtl_VSTS_GeneratedVhdStorageAccount.png)

> The caveat here is that it is not always the first storage account in the resource group that contains the generated vhd container, it could be in the other storage acconts in the resource group.

# How to identify the container the custom image needs to be copied into?  
The easiest way to identify which conatiner the custom image needs to go into is by clicking the add custom image option from the dev test lab settings blade. As demonstrated in the image below, the powershell is generated for you, simply reference the name of the storage account from here. 

![AzureDevTestLab](/images/screenshots/tarun/AzureDTL/AzureDtl_VSTS_GenerateCustomImageOption.png)

> The storage account listed in the auto generated powershell in the add custom image window is the one your custom images need to go into.    

# Copying Custom Images between Azure DevTest Labs
Now that we have successfully identfied the storage account and container that has the custom images, we can use the AzCopy command to copy the custom images between the blobs in the storage accounts without having to download the custom image locally first. More details on the AzCopy command can be found in the [Azure MSDN Reference](https://azure.microsoft.com/en-gb/documentation/articles/storage-use-azcopy/)

 ![AzureDevTestLab](/images/screenshots/tarun/AzureDTL/AzureDtl_CopyCustomImageBetweenTwoDevTestLabs.png)

The AzCopy command has the following syntax for copying a blob between two storage accounts... 

``` cmd  
AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 
       /Dest:https://destaccount.blob.core.windows.net/mycontainer2 
       /SourceKey:key1 
       /DestKey:key2 
       /Pattern:abc.txt
```

In our scenario source is `storage account 1` destination is `storage account b`. The secure key's for these can be found in the `Access Keys` window accessible from the storage account settings blade. 

Open a command window and navigate to the AzCopy installation directory on your computer - where the AzCopy.exe executable is located. If desired, you can add the AzCopy installation location to your system path. By default, AzCopy is installed to `%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy` (64-bit Windows) or `%ProgramFiles%\Microsoft SDKs\Azure\AzCopy` (32-bit Windows).

![AzureDevTestLab](/images/screenshots/tarun/AzureDTL/AzureDtl_Task_AzCopyBetweenTwoStorageAccounts.png)

The copy operation took 40 minutes for 127 GB worth of data.

![AzureDevTestLab](/images/screenshots/tarun/AzureDTL/AzureDtl_AzCopy_Summary.png)

Navigate into the dev test lab 2 and click on add custom images, you'll see the MyCustomImage.vhd show up under the section `add existing images`. Voila!

![AzureDevTestLab - Add custom image](/images/screenshots/tarun/AzureDTL/AzureDtl_ImportCustomImages.png)

Recommended Reading... 

- [Why care about DevOps?](http://www.visualstudiogeeks.com/blog/devops/marry-cloud-and-devops-enterprise-devops-is-for-real)
- [What is Technical Debt & why is it a problem?](http://www.visualstudiogeeks.com/blog/sonarqube/devops/Configure-TFS2015-with-SonarQube-using-BuildTask-to-Track-Technical-Debt)
- [How to install SonarQube as a Windows Service with SQL (Windows Auth) for code analysis with VSTS](http://www.visualstudiogeeks.com/blog/DevOps/Install-SonarQube-As-WindowsService-With-SQLServer-WindowsAuth-VSTS-TeamBuild)

Hope you found this useful...  

Tarun