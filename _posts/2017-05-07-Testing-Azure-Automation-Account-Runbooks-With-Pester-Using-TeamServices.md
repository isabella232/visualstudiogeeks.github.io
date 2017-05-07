---
layout: post
title: "Azure Automation - Testing PowerShell Runbooks with Pester using Team Services"
date: 2017-05-07
author: tarun
tags: ["DevOps", "AzureAutomation", "VSTS", "Pester"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
description: "Testing Azure Automation runbooks stored in GitHub through Pester in VSTS"
permalink: /DevOps/TestingAzureAutomationPowerShellRunbooksWithPesterInTeamServices
published: true
keywords: "DevOps, Azure, AzurePortal, AzureAutomation, PowerShell, Runbooks, Pester, Team Services, TFS, VSTS, Infrastructure As Code"
---
Azure Automation is a hosted, managed Service that allows you to automate application life cycle areas such as server provisioning and server configuration management. Chances are that if you are already using Azure Automation, you have runbooks that help automate the routine operational tasks. In this blogpost I'll show you how to leverage the integration between Azure Automation & GitHub to version control your runbooks. In addition to this we'll see how easy it is to create unit tests for your runbooks using Pester and then creating a CI pipeline for your runbooks using Team Services. 
<!--more--> 

# Introduction 
Create an Azure Automation account if you don't already have one. In this blog post I am going to use the Azure Automation account `geeks-demo-westeurope-auto`. To set up version control integration for the runbooks created in this automation account, select the `source control` under the section `account settings` on the settings blade. Work through the options to connect your repository by authorizing the account access, selecting the repository, specifying the branch and selecting the folder where the runbooks would be kept. 

![Integrate Azure Automation to GitHub](/images/screenshots/tarun/AzureAutomation/IntgerateAzureAutomationToGitHub.PNG)
 
Once the integration between the Azure Automation account and the GitHub repository is completed, you should see an option to sync the repository with the Azure Automation account. 

![Sync Azure Automation Account with GitHub](/images/screenshots/tarun/AzureAutomation/SyncGitHubRepoWithAzureAutomationAccount.PNG)

> When you click Sync you would be warned that all runbooks in this folder would be synced to this Azure Automation account, simply accept yes to continue. 

Once the sync is complete, you'll see all your runbooks in the runbook section in the Automation Account. 

> Note: *The runbooks would be added in the new status* and would need to be published manually. 

# Sample script
In this blogpost, I am going to create a simple runbook that just introduces a new function to add two numbers. Go through the flow of Azure Automation to add a new runbook and select the type as 'PowerShell'.  

![Sample runbook to add two numbers](/images/screenshots/tarun/AzureAutomation/AddTwoNumbersRunbook.PNG)

Now copy the contents of the script given below...

``` powershell 
param(){
    [parameter(Mandatory=true)]
    [int]$no1,
    [parameter(Mandatory=true)]
    [int]$no2
}

function Sum ([int]$no1, [int]$no2){
	return $no1 + $no2
}

$result = Sum $no1 $no2

Write-Output "The sum of $($no1) and $($no2) is $($result)"
```

Now click save and then click Commit to push these changes into your GitHub repository.

![Commit runbook to GitHub](/images/screenshots/tarun/AzureAutomation/CommitRunbookToGitHubByClickingCheckIn.PNG)

> You should now see the runbook in your repository in GitHub

# Adding Unit Test
In order to add a test for this repository, go to GitHub and add the following file in the repository. 

``` PowerShell
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
. "$here\calculator.ps1"

Describe -Tags "Demo" "Sum"{
	It "adds positive numbers" {
		Sum 2 3 | Should be 5
	}
}

Describe -Tags "Demo" "Sum"{
	It "adds a positive and a negative number" {
		Sum 2 -3 | Should be -1
	}
}

Describe -Tags "Demo" "Sum"{
	It "adds two negative number" {
		Sum -2 -3 | Should be -5
	}
}

Describe -Tags "Demo" "Sum"{
	It "adds two zero numbers" {
		Sum 0 0 | Should be 0
	}
}
```

# Create a pipeline to test AzureAutomation Runbook via Team Services 

From the build hub in Visual Studio Team Services select an empty template. Configure the source task to point to the GitHub repository that holds the scripts for run books that you use in the Automation Account. 

![Build Pipeline Get Sources from GitHub](/images/screenshots/tarun/AzureAutomation/BuildPipelineGetSources.png)

Next add the task for Pester, if you don't already have the Pester task installed in your Visual Studio Team Services account, you can download the free open source extension from the Visual Studio Team Services MarketPlace.

[Pester Task from MarketPlace](https://marketplace.visualstudio.com/items?itemName=richardfennellBM.BM-VSTS-PesterRunner-Task)

Configure the extension to the folder that contains the unit tests...  

![Configure Pester](/images/screenshots/tarun/AzureAutomation/BuildPipelinePesterTaskConfigurationExample.png)

You should include the Publish Test Results task into the pipeline in order to publish the test results generated by the test execution to be loaded up into Team Services. Team Services will then be able to render the test results in the test result & analysis screens along side the build summary. 

Next add the task for GitVersion in the pipeline, if you don't already have the GitVersion extension in your Team Serviecs account, you can download the free open source extension from the MarketPlace. 

[GitVersion Task in MarketPlace](https://marketplace.visualstudio.com/items?itemName=gittools.gitversion)

If you are wondering what the benefits of GitVersion are, stay tuned I'll be writing a blogpost on that shortly. 

Now, trigger the build and see Team Services run up test execution for your automation account PowerShell scripts from GitHub... 

![Configure Pester](/images/screenshots/tarun/AzureAutomation/BuildPipelineForPowerShellPesterUnitTestExecution.PNG)

# Summary 

![Configure Pester](/images/screenshots/tarun/AzureAutomation/BuildPipelinePesterTestExecutionResults.PNG)

Like what you see? Feel free to share... Give me a shout on twitter [@arora_tarun](twitter.com/arora_tarun) if you run into any issues.... 

 