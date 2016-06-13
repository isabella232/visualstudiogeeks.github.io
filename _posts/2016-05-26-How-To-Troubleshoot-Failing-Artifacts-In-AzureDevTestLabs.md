---
layout: post
title: "How to troubleshoot failing Artifacts in AzureDevTestLabs"
date: 2016-06-12 20:00:00 
author: tarun 
tags: ["DevOps", "Azure", "AzureDevTestLabs"]
categories:
- DevOps
img: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
permalink:  /blog/DevOps/How-to-troubleshoot-failing-artifacts-in-AzureDevTestLabs
description: "DevOps AzureDevTestLabs - How to troubleshoot & debug failing artifacts in AzureDevTestLabs"
keywords: "DevOps, Azure, AzureDevTestLabs"
---
If you are using Azure DevTestLabs you are probably loving and leveraging the Public/Private artifact repository feature available in the labs. The feature is great but should the artifacts fail it's not easy to get to the logs of the failure to diagnose the issue. In this blogpost I'll show you the two available ways to see the artifact logs in Azure DevTestLabs. 

<!--more--> 

# Troubleshooting Artifacts in AzureDevTestLabs
To view the logs related to your custom artifact you have 2 options:

![AzureDevTestLabs InfrastructureIsCode](/images/screenshots/tarun/AzureDTL/AzureDtl_ArtifactDeploymentFailedWhy.png)

- __Azure Portal__: To view the artifact logs from the Azure Portal follow these steps, 
    - In the Azure Portal from under dev test labs, click the virtual machine you want to investigate
    - For this virtual machine, from under the essential section click on the Resource Group for the virtual machine
    - From the resource group click on the virtual machine 
    - From the settings blade select Extensions 
    - This will show you the extensions installed or being installed on the VM
    - Click on the failed extension to see the detailed status 

      ![AzureDevTestLabs InfrastructureIsCode](/images/screenshots/tarun/AzureDTL/AzureDtl_Artifact-ExtensionFailureLog.png)

- __Logs on the VM__: To view the artifact logs from with in the virtual machine follow these steps, 
    - Log into the Virual Machine
    - Navigate to the folder `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Status\0.status`
       
      ![AzureDevTestLabs InfrastructureIsCode](/images/screenshots/tarun/AzureDTL/AzureDtl-StatusFailedArtifactLogFIleS.png)

# Other posts on Azure DevTestLabs
You might find these other posts on Azure DevTestLabs useful.

- [Deploy new VM in an existing AzureDevTestLab using VSTS](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS)
- [Copy custom images (VHD) between AzureDevTestLabs](http://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs)
- [Configure WinRm with ARM template using PowerShell artifact](http://www.visualstudiogeeks.com/blog/DevOps/Configure-winrm-with-ARM-template-in-AzureDevTestLab-VM-deployment-using-PowerShell-artifact)
- [Continuous Deployment with AzureDevTestLabs](http://www.visualstudiogeeks.com/blog/DevOps/Use-VSTS-ReleaseManagement-to-Deploy-and-Test-in-AzureDevTestLabs)

I hope you find this useful. 

Happy troubleshooting!
