---
layout: post
title: "Join a VM to existing AD Domain using ARM template in Azure Dev Test Lab"
date: 2016-05-17 12:00:00 
author: tarun 
tags: ["DevOps", "Azure", "AzureDevTestLabs", "ARM"]
categories:
- blog                #important: leave this here
- "DevOps"
permalink:  /blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs
description: "DevOps Azure DevTestLab InfrastructureAsCode VSTS AzureResourceManager InfrastructureAutomation VHD CustomImages DomainJoin ActiveDirectory"
image:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /images/screenshots/thumbs/
---
Learn how to domain join your Azure DevTestLab VM to with an Active Directory Domain Controller using a powershell artifact. We'll trigger this process from VSTS. The private artifact repository will also be available & exposed in DevTestLab for virtual machines in the lab.    
<!--more--> 

# Scenario 
While provisioning a virtual machine in the AzureDevTest Lab you would like the newly provisioned virtual machine to be joined up to an existing Active Directory Domain Controller. This can be achieved by running a PowerShell script, that is wrapped up as an artifact. This artifact can be exposed in the DevTestLab via a private artifact repository. 

![AzureDevTestLabs Join Domain Scenario]({{site.url}}/images/screenshots/tarun/AzureDTL/AzureDtl_JoinDomainTask.png)

In this blogpost I'll show you how you can leverage a private artifact repository which intern uses powershell script to join your newly created Azure virtual machine in Azure DevTest Lab to an existing active directory domain.

# Domain Join Artifact
The JoinDomain PowerShell script and Artifactfile json file is available for download on [my gitHub repository](https://github.com/tarunaroraonline/azure-devtestlab/tree/master/Artifacts/windows-joinDomain)

- [Artifactfile.json](https://github.com/tarunaroraonline/azure-devtestlab/blob/master/Artifacts/windows-joinDomain/Artifactfile.json)
- [JoinDomain.ps1](https://github.com/tarunaroraonline/azure-devtestlab/blob/master/Artifacts/windows-joinDomain/JoinDomain.ps1)

Download the artifact and use the [instructions here](https://azure.microsoft.com/en-gb/documentation/articles/devtest-lab-add-artifact-repo/) to set up your private artifact repository. DevTestLabs support both Git and VSTS as artifact repository endpoints... 

Once you have the private artifact repository set up and the Join Domain Artifact added, in the Azure Portal you should see something like this... 

![AzureDevTestLabs Join Domain Scenario]({{site.url}}/images/screenshots/tarun/AzureDTL/AzureDtl_AzCopy_CustomArtifactRepoJoinDomainArtifact.png)

# Plug in Domain Join Artifact to your ARM template 
We'll see how easy it is to add this private artifact script into your Azure DevTestVM ARM template. If you don't already have an ARM template then refer to [my blogpost here](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS) that shows you how to achieve this. 

![AzureDevTestLab ARM Template]({{site.url}}/images/screenshots/tarun/AzureDTL/ExportArmTemplateFromDevVmInDevTestLab.png)

In your AzureDevTestLab ARM template add the following three parameters...

```shell
    /* Join Domain Parameters */
        "Join_Domain_Domain": {
        "type": "string",
        "defaultValue": "myDomain.net"
        },
        "Join_Domain_UserName": {
        "type": "string"
        },
        "Join_Domain_Password": {
        "type": "string"
``` 
Supplement the artifact section of the template with the following artifact... 

```shell
        "artifacts": 
        [          
            /* Join Domain */
            {
                "artifactId": "[resourceId('Microsoft.DevTestLab/labs/
                        artifactSources/artifacts', parameters('labName'), 
                        'privaterepo170', 'JoinDomain')]",
                        
                "parameters": 
                [
                    {
                        "name": "Domain",
                        "value": "[parameters('Join_Domain_Domain')]"
                    },
                    {
                        "name": "UserName",
                        "value": "[parameters('Join_Domain_UserName')]"
                    },
                    {
                        "name": "Password",
                        "value": "[parameters('Join_Domain_Password')]"
                    }
                ]
            }
        ]
```

Commit the changes to the repository... 

# Provision a new virtual machine in DevTestLab & trigger Join Domain artifact

> I will use VSTS to trigger the deployment of the Azure DevTestLab ARM template. If you don't want to use VSTS, you could deploy from this template directly from the Azure portal. In case you want to use VSTS to [deploy a new Virtual Machine in an existing Azure DevTestLab then follow these instructions](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS)

Ammend the build definition to include the values for the three variables added to the ARM template. 

![VSTS AzureDevTestLab trigger new VM deloyment]({{site.url}}/images/screenshots/tarun/AzureDTL/AzureDtl_JoinDomain_VSTS_Variables.png)

Ensure that you add these variables into the template parameters section of the task. 

![VSTS AzureDevTestLab trigger new VM deloyment]({{site.url}}/images/screenshots/tarun/AzureDTL/AzureDtl_JoinDomain_Vsts_Task.png)
 
Trigger a new build to create a new VM in an Azure DevTestLab to provision a new VM that runs the private artifact to domain join the newly provisioned virtual machine...  

# Validate that the JoinDomain artifact worked successfully
To validate the execution of the JoinDomain artifact navigate to the resource group of the newly deployed virtual machine. Click on deployments from the settings blade to load the history of all deployments that have taken place against this resource group. 

![VSTS AzureDevTestLab trigger new VM deloyment]({{site.url}}/images/screenshots/tarun/AzureDTL/AzureDtl_JoinDomainArtifact_Validate.png)
 
The deployments blade will show you the full history of all deployments as well as the details of the artifacts run up against the resource group. You also have the ability to rerun the deployments of the artifacts from here. As you can see in the below screen shot, it is also possible to see the actual parameters passed to the artifact. 

![VSTS AzureDevTestLab trigger new VM deloyment]({{site.url}}/images/screenshots/tarun/AzureDTL/AzureDtl_ArtifactDeploymentDetails.png)

__Voila!__ Now that the artifact has successfully been run, you'll see the virtual machine registered in active directory under the computers OU. You will also be able to use domain credentials to log into the virtual machine.

![VSTS AzureDevTestLab trigger new VM deloyment]({{site.url}}/images/screenshots/tarun/AzureDTL/AzureDtl_AD_Domain.png) 

Check out other posts on AzureDevTest labs:

- [Deploy new VM in an existing AzureDevTestLab using VSTS](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS)
- [Copy custom images (VHD) between AzureDevTestLabs](http://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs)
- [Configure WinRm with ARM template using PowerShell artifact](http://www.visualstudiogeeks.com/blog/DevOps/Configure-winrm-with-ARM-template-in-AzureDevTestLab-VM-deployment-using-PowerShell-artifact)

Happy Deployments! 

Tarun 