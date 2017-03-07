---
layout: post
title: "Azure Automation - Manage AWS EC2 with Azure Automation Pull Server"
date: 2017-03-07
author: tarun
tags: ["DevOps", "Azure", "AzureAutomation"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
description: "Use Azure Automation DSC Pull Server to manage Amazon EC2 Instances"
permalink: /DevOps/ManageAwsEC2InstancesWithAzureAutomationDscPullServer
keywords: "DevOps, Azure, AzurePortal, AzureAutomation, DSC Pull Server, AWS EC2, PowerShell"
---

In this blog post I'll show you how to manage the desired state configuration of a server in Amazon cloud using Azure Automation DSC Pull Server... My goal is to hook up the virtual machines I have in AWS cloud to use the Azure Automation DSC Pull server and ensure compliance to the BasicWebServer script which checks compliance for a web server feature on the node machine.  
<!--more-->

# What is Azure Automation?
Azure automation is a managed Service offered through Azure to script, test and automate application life cycle areas such as environment provisioning and server configuration… Azure automation helps you automate the manual, long-running, error-prone, and frequently repeated tasks that are commonly performed in a cloud and enterprise environment. It saves time and increases the reliability of regular administrative tasks and even schedules them to be automatically performed at regular intervals all from within the very familiar interface of azure portal... 

![WhatIsAzureAutomation](/images/screenshots/tarun/AzureAutomation/WhatIsAzureAutomation.PNG)

# Why Azure Automation
In the past most of the automation has been focussed on Enterprise data centre, IT Operations and IT Processes… The movement to the cloud is opening up a new suite of scenarios for automation to work against… Such as,
- Managing hybrid cloud that involves on-premise and azure 
- Multi vendor cloud scenarios such as Azure and Aws
- Heterogeneous cloud like windows and Linux… 
- There is a massive push to enable self-service of IT offerings and of course now DevOps

> The old ways of management do not necessarily work at scale of cloud… Modern cloud needs modern management… *Azure Automation* is build keeping these complex scenarios in mind...

# Manage AWS EC2 from Azure Automation
I've created a free subscription in Amazon AWS and spun up a couple of instances of Windows 2016 basic virtual machine. My goal is to hook these machines with the Azure Automation 

![AzureAutomation-UpdateAzureModules](/images/screenshots/tarun/AzureAutomation/AwsEc2InstancesInAmazonConsole1.PNG) 

Next up login to the instance and ensure that it has Windows Management Framework 5 installed, you can test this by launching PowerShell and executing the following command `$host.version`

![AzureAutomation-AwsEc2CheckPowerShellWmFramework](/images/screenshots/tarun/AzureAutomation/AmazonEC2CheckWMF5Exists.PNG)

In order to configure the LCM on the AWS EC2 server you need to run up the following script on the EC2 machine, this script basically generates the MOF file needed to register this virtual machine with Azure Automation

``` PowerShell
 # The DSC configuration that will generate metaconfigurations
 [DscLocalConfigurationManager()]
 Configuration DscMetaConfigs
 {

     param
     (
         [Parameter(Mandatory=$True)]
         [String]$RegistrationUrl,
         [Parameter(Mandatory=$True)]
         [String]$RegistrationKey,
         [Parameter(Mandatory=$True)]
         [String[]]$ComputerName,
         [Int]$RefreshFrequencyMins = 30,
         [Int]$ConfigurationModeFrequencyMins = 15,
         [String]$ConfigurationMode = "ApplyAndMonitor",
         [String]$NodeConfigurationName,
         [Boolean]$RebootNodeIfNeeded= $False,
         [String]$ActionAfterReboot = "ContinueConfiguration",
         [Boolean]$AllowModuleOverwrite = $False,
         [Boolean]$ReportOnly
     )

     if(!$NodeConfigurationName -or $NodeConfigurationName -eq "") {
         $ConfigurationNames = $null
     }
     else{
         $ConfigurationNames = @($NodeConfigurationName)
     }

     if($ReportOnly){
     $RefreshMode = "PUSH"
     }
     else{
     $RefreshMode = "PULL"
     }

     Node $ComputerName {
         Settings
         {
             RefreshFrequencyMins = $RefreshFrequencyMins
             RefreshMode = $RefreshMode
             ConfigurationMode = $ConfigurationMode
             AllowModuleOverwrite = $AllowModuleOverwrite
             RebootNodeIfNeeded = $RebootNodeIfNeeded
             ActionAfterReboot = $ActionAfterReboot
             ConfigurationModeFrequencyMins = $ConfigurationModeFrequencyMins
         }

         if(!$ReportOnly){
         ConfigurationRepositoryWeb AzureAutomationDSC
             {
                 ServerUrl = $RegistrationUrl
                 RegistrationKey = $RegistrationKey
                 ConfigurationNames = $ConfigurationNames
             }
             ResourceRepositoryWeb AzureAutomationDSC
             {
             ServerUrl = $RegistrationUrl
             RegistrationKey = $RegistrationKey
             }
         }
         ReportServerWeb AzureAutomationDSC {
             ServerUrl = $RegistrationUrl
             RegistrationKey = $RegistrationKey
         }
     }
 }

 # Create the metaconfigurations
 # TODO: edit the below as needed for your use case
 $Params = @{
     RegistrationUrl = '<fill me in>';
     RegistrationKey = '<fill me in>';
     ComputerName = @('<some VM to onboard>', '<some other VM to onboard>');
     NodeConfigurationName = 'SimpleConfig.webserver';
     RefreshFrequencyMins = 30;
     ConfigurationModeFrequencyMins = 15;
     RebootNodeIfNeeded = $False;
     AllowModuleOverwrite = $False;
     ConfigurationMode = 'ApplyAndMonitor';
     ActionAfterReboot = 'ContinueConfiguration';
     ReportOnly = $False;  # Set to $True to have machines only report to AA DSC but not pull from it
 }

 # Use PowerShell splatting to pass parameters to the DSC configuration being invoked
 # For more info about splatting, run: Get-Help -Name about_Splatting
 DscMetaConfigs @Params
```

> You can find the Azure Automation endpoint and the primary key in Azure Automation 'Key' under __account settings__, screen shot below...

![AzureAutomation-AzureAutomationAccountSettingsKey](/images/screenshots/tarun/AzureAutomation/AzureAutomationAccountSettingsKey.PNG)

Now trigger this script in Azure PowerShell... This will generate the MOF file for this server...

![AzureAutomation-AwsEc2AzureAutomationDscMetaConfig](/images/screenshots/tarun/AzureAutomation/AwsEc2AzureAutomationDscMetaConfig.PNG)

The LCM configuration should now be updated... You can check it by running the following command `Get-DscLocalConfigurationManager`

![AzureAutomation-AmazonEC2LcmDscConfiguration](/images/screenshots/tarun/AzureAutomation/AmazonEC2LcmDscConfiguration.PNG)

Now invoke the MOF file to register the Agent with Azure Automation... `Set-DscLocalConfigurationManager -Path <path>\DscMetaConfigs -ComputerName <ComputerName>`

![AzureAutomation-AwsEc2RegisterWithAzureAutomation](/images/screenshots/tarun/AzureAutomation/AwsEc2RegisterWithAzureAutomation.PNG)

At this point AWS EC2 instance should be registered to the DSC Pull server in Azure Automation, you can see this by browsing to Azure Automation in the Azure Portal...

![AzureAutomation-AwsEc2AzureAutomationRegisteredNode](/images/screenshots/tarun/AzureAutomation/AwsEc2AzureAutomationRegisteredNode.PNG)

Now next time the node synchronizes with the DSC Pull server it will apply the DSC Configuration BasicWebSerer which will set up the windows Web Server feature on the node... 

# Summary

Now Azure Automation allows you to not only leverage your automation scripts on Azure but on heterogeneous cloud environments... Managing Multiple Cloud providers is a breeze... 

Hope you find this useful... 

Tarun 



