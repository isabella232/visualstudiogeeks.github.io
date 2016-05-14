---
layout: post
title: "Configure WinRm with ARM template in AzureDevTestLab VM deployment using PowerShell artifact"
date: 2016-05-14 12:05:00 
author: Tarun Arora 
tags: ["DevOps", "Azure", "AzureDevTestLabs", "ARM", "WinRM"]
categories:
- blog                #important: leave this here
- "DevOps"
permalink:  /blog/DevOps/Configure-winrm-with-ARM-template-in-AzureDevTestLab-VM-deployment-using-PowerShell-artifact
description: "DevOps Azure DevTestLab InfrastructureAsCode VSTS AzureResourceManager InfrastructureAutomation VHD CustomImages"
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /images/screenshots/thumbs/
---
WinRM configuration isn't straightforward, it is tedious to say the least, if you get one step in the process wrong, more often than not it comes back to bite you later. In this blogpost I'll show you a really cool way to automate WinRM configuration through Azure Resource Manager (ARM) template. Further in this blog post, I'll show you how to trigger the deployment of this ARM template to create a new VM in AzureDevTestLab using VSTS.    
<!--more--> 

> In return I ask you give `@arora_tarun` & `@onlyutkarsh` a shout on twitter if you find this blogpost useful...  

Other WinRM configuration blogposts... 

- _[Instructions on How to configure WinRM for HTTP access on a domain joined environment](http://www.visualstudiogeeks.com/blog/teamfoundationserver/HowToConfigureTFSMachineGroupsOnPremAndUseWithTFBuildRelease)_
- _[Instructions on How to configure WinRM for HTTPS manually (coming soon)]_
 

# Scenario 
The scenario this blogpost targets to address is to automate the following using an ARM template,

- Configuration of WinRM
- Generation of a local certificate 
- Using the newly generate certificate to register an Https listner 
- Firewall configuration to allow traffic on https through port 5986 
  
![AzureDevTestLab WinRM ARM scenario](/images/screenshots/tarun/AzureDTL/AzureDtl_WinRmArmScenario.png)

The Visual Studio Team Services (VSTS) Hosted agent can be used to run a build job. If your build pipeline includes a task to run a remote powershell script. The VSTS agent requires that the target machine has WinRM https listner configured. I am assuming that the target machine is a standalone machine, not joined to a domain. The pre-requisites to allow communication between the hosted agent and the target workgroup based Azure virtual machine is as listed on [MSDN](https://msdn.microsoft.com/en-us/Library/vs/alm/Build/steps/deploy/powershell-on-target-machines)... 

| Target Machine state | Trust with automation agent | Machine Identity | Auth Account | Auth Mode | Auth Account permission on target machine | Conn Type|
| ------------- | ------------- |----|---|---|---|---|
| Workgroup machine in Azure | Un Trusted  | DNS name | Local machine account | NTLM | Machine admin | WinRM HTTPS | 

Pre-requisites in Target machine for Deployment Task to succeed,

- WinRM HTTPS port (default 5986) opened in Firewall.
- Disable UAC remote restrictions
- Credential in `<MachineName>\<Account>` format.
- Trusted certificate in Automation agent.
- If Trusted certificate not in Automation agent then Test Certificate option enabled in Task for deployment.

# Prepare ARM template for WinRM configuration
The Azure Quick Start templates repository has a WinRM configuration script, [view script](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/ConfigureWinRM.ps1)    

![AzureDevTestLab WinRM ConfigurationScript](/images/screenshots/tarun/AzureDTL/AzureDtl_WinRm_ConfigurationScript.png)

This script has dependencies on two other script, links to all three resources are listed below,
 
- [ConfigureWinRM](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/ConfigureWinRM.ps1)
- [Make Certificate](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/makecert.exe)
- [Trigger configuration on Target Machine](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/winrmconf.cmd)

Now that we have the scripts we need to automate the winRm configuration, we'll look at how to plug these into the DevTestLab ARM template...

> You need the __Azure DevTestLab ARM template__. If you don't already have the Azure DevTestLab ARM template, then follow the steps in [my previous blogpost](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS) to retrive the DevTestLab ARM template.

![AzureDevTestLab WinRM ConfigurationScript](/images/screenshots/tarun/AzureDTL/ExportArmTemplateFromDevVmInDevTestLab.png)

In the parameters section of your ARM template include two parameters... 

- __Run_Powershell.scriptFileUris__: These are links to the WinRM configuration scripts.
- __Run_Powershell.scriptToRun__: Name of the script to run (entry point). 


```json
/* WinRM Custom Script Parameters */
     "Run_PowerShell.scriptFileUris": {
      "type": "string",
      "defaultValue": "[[\"https://raw.githubusercontent.com
                                /Azure/azure-quickstart-templates
                                /master/201-vm-winrm-windows
                                /ConfigureWinRM.ps1\", 
                        \"https://raw.githubusercontent.com
                                /Azure/azure-quickstart-templates
                                /master/201-vm-winrm-windows
                                /makecert.exe\", 
                        \"https://raw.githubusercontent.com
                                /Azure/azure-quickstart-templates
                                /master/201-vm-winrm-windows
                                /winrmconf.cmd\"]"
    },
    "Run_PowerShell.scriptToRun": {
      "type": "string",
      "defaultValue": "ConfigureWinRM.ps1"
    }
```

In the variables section of your ARM template include a variable for specifying the hostName of the target VM. The host name specified here is used to create the self signed local certificate for Https listner. This parameter will translate to `*.northeurope.cloudapp.azure.com` you should alternatively chose a different convention if your azure VM is domain joined or has a different fdqn format. 

```json
    "hostDNSNameScriptArgument": 
        "[concat('*.',resourceGroup().location,'.cloudapp.azure.com')]"
``` 

In the artifact section of the ARM template use the powershell artifact to make a call to your powershell script by passing the variable parameter required by the script. 

```json
"artifacts": 
        [
          /* Configure WinRm using Windows Run Powershell artifact */
          {
            "artifactId": 
                "[resourceId('Microsoft.DevTestLab/labs/artifactSources/artifacts', 
                        parameters('labName'), 'Public Repo', 
                                'windows-run-powershell')]",
            "parameters": [
              {
                "name": "scriptFileUris",
                "value": "[parameters('Run_PowerShell.scriptFileUris')]"
              },
              {
                "name": "scriptToRun",
                "value": "[parameters('Run_PowerShell.scriptToRun')]"
              },
              {
                "name": "scriptArguments",
                "value": "[variables('hostDNSNameScriptArgument')]"
              }
            ]
          }
        ]
```

Save the changes to the template. Your template is now ready to automatically configure WinRM... 

> I will use VSTS to trigger the deployment of the Azure DevTestLab ARM template. If you don't want to use VSTS, you could deploy from this template directly from the Azure portal. In case you want to use VSTS to [deploy a new Virtual Machine in an existing Azure DevTestLab then follow these instructions](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS)

Trigger a new build to create a new VM in an Azure DevTestLab to provision a new VM preconfigured with WinRM...

![VSTS AzureDevTestLab trigger new VM deloyment](/images/screenshots/tarun/AzureDTL/AzureDtl_Task_VSTSBuildToCreateNewVM.png)

Wait for the build process to complete, once the build succeeds, you can see the Create Azure DevTest Labs VM deployment task has successfully completed. 

![VSTS AzureDevTestLab trigger new VM deloyment](/images/screenshots/tarun/AzureDTL/AzureDtl_Task_VSTSBuildDeploymentCompleted.png)

# Validate WinRM configuration on Azure VM
Login to the newlyt deployed VM, let's validate if WinRM has been configured correctly... 

- __WinRM Listners__: 

   Launch `PowerShell` in administrator mode... 
   
```shell
    winrm e winrm/config/listener 
```

![VSTS AzureDevTestLab WinRM Https Listner](/images/screenshots/tarun/AzureDTL/AzureDtl_VSTS_WinRM_HttpsListner.png)

- __Certificate__:

     Open up the local certificate store in the machine and navigate to the  

![VSTS AzureDevTestLab WinRM Self Signed Certificate](/images/screenshots/tarun/AzureDTL/AzureDtl_WinRMCertificateStore.png)

- __Firewall__: 
 
     Open up Advanced firewall settings and look for an inbound firewall exception rule set to allow HTTPS TCP access on port 5986

![VSTS AzureDevTestLab WinRM Self Signed Certificate](/images/screenshots/tarun/AzureDTL/AzureDtl_WinRM_HttpsTcp5986FirewallException.png)

Voila you are all set! 

# Trigger Remote PowerShell script from VSTS 
Now run a powershell remote script using hosted VSTS agent, make sure you have the `test certificate` check box checked and the user credentials of the local administrator account are specified in `machinename\username` format. 

![VSTS AzureDevTestLab WinRM Self Signed Certificate](/images/screenshots/tarun/AzureDTL/AzureDtl_VSTS_RunPowerShellOnTargetMc_Config.png)

Trigger a build and wait for it to complete successfully... 

![VSTS WinRM Remote PowerShell execution](/images/screenshots/tarun/AzureDTL/AzureDtl_RemotePowerShellExecutionVSTS.png)

Just in case you hit the following issue... 

`2016-05-13T10:43:17.0084649Z ##[error]Connecting to remote server dtlmachinename5.northeurope.cloudapp.azure.com 
failed with the following error message : Access is denied. For more information, see the about_Remote_Troubleshooting Help topic.`

[More details on this issue here](http://stackoverflow.com/questions/22680211/access-denied-on-remote-winrm/22817331#22817331)
 
You can get around this issue by running the following Powershell script on the target machine... 

```shell
Set-PSSessionConfiguration -ShowSecurityDescriptorUI -Name Microsoft.PowerShell
```

Check out other posts on AzureDevTest labs

- [Deploy new VM in an existing AzureDevTestLab using VSTS](http://www.visualstudiogeeks.com/blog/DevOps/Deploy-New-VM-To-Existing-AzureDevTestLab-From-VSTS)
- [Copy custom images (VHD) between AzureDevTestLabs](http://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs)

Happy Deployments! 

Tarun 