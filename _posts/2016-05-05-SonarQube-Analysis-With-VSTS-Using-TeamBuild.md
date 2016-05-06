---
layout: post
title: "SonarQube Analysis in Continuous Integration using Team Build in VSTS"
date: 2016-05-05 13:00:00 
author: Tarun Arora 
tags: ["DevOps", "SonarQube"]
categories:
- blog                #important: leave this here
- "DevOps"
permalink:  /blog/DevOps/SonarQube-Analysis-With-VSTS-Using-TeamBuild
description: "DevOps Technical Debt CodeAnalysis TeamBuild C# SonarQube CI VSTS (constantly updated)"
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /images/screenshots/thumbs/
---
In this blogpost we'll learn how to expose SonarQube endpoint to Team Build in VSTS and run a full code analysis cycle as part of the continuous integration pipeline. We'll then look at the code analysis results summary in VSTS and see the build version mapping between VSTS and SonarQube.
<!--more--> 

# Pre-requisites 
In the last blogpost we set up a SonarQube with SQL Server (Win Auth) from scratch. This blogpost builds on top of that, if you don't have a SonarQube server configured yet, follow the instructions here before moving forward [http://www.visualstudiogeeks.com/blog/DevOps/Install-SonarQube-As-WindowsService-With-SQLServer-WindowsAuth-VSTS-TeamBuild](http://www.visualstudiogeeks.com/blog/DevOps/Install-SonarQube-As-WindowsService-With-SQLServer-WindowsAuth-VSTS-TeamBuild).

The blog post also requires that you have a VM set up with VSTS Build Agent. If you don't already have a build agent configured on a seperate VM, then follow the instructions [https://msdn.microsoft.com/en-us/library/vs/alm/build/agents/windows](https://msdn.microsoft.com/en-us/library/vs/alm/build/agents/windows) to set up a VSTS Agent. The Agent needs to have jave installed. SonarQube requires that the Visual Studio Agent has Java installed. If you do not have Java installed on the Agent, expect to see the following error message...

![SonarQube Download Page](/images/screenshots/tarun/SonarQube/VSTS_ErrorMessageWhenAgentDoesntHaveJavaInstalled.png) 

If you intent to use the hosted agent in VSTS, then you don't need a machine with VSTS Agent set up. 

# Infrastructure Setup 
SonarQube is setup using a single VM configuration where both the web server and the database are on one single machine. The VSTS Agent is set up on another virtual machine. For VSTS Build Agent to consume SonarQube endpoint for code analysis, the Agent needs access to SonarQube. Since SonarQube is currently configured on port 9000, the firewall rules need to be ammended on both SonarQube server and VSTS Agent to allow communication between the two boxes on TCP port 9000.  

![SonarQube Download Page](/images/screenshots/tarun/SonarQube/sonarQubeIntegrationWithVSTSAgent.png) 

Once you have added the firewall exception to allow communication from SonarQube to VSTS Agent and visa versa on TCP port 9000, launch cmd on both the machines and telnet on port 9000 to validate the channel. 

# Configuring SonarQube Analysis in a build pipeline
Launch Visual Studio Team Services and navigate to the Build Hub. Click the + icon to create a new build definition. Choose the 'Visual Studio' build template.  

![SonarQube Download Page](/images/screenshots/tarun/SonarQube/VSTS_VisualStudioBuildTemplate.png) 

The build template adds a bunch of tasks to the newly created build definition. 

![SonarQube Download Page](/images/screenshots/tarun/SonarQube/VSTS_VisualStudioTemplateDefaultTasks.png) 

Click on the + Add build step link. This will open up the task gallery. From the Build tab Add the task 'SonarQube for MSBuild - Begin Analysis' and 'SonarQube for MSBuild - End Analysis' task. Drag the Begin Analysis task to before the Build solution step and the end analysis task to after the unit test execution task. The build pipeline should look as follows now... 

![SonarQube Download Page](/images/screenshots/tarun/SonarQube/VSTS_AddSonarQubeBeginEndAnalysisTasks.png) 

We need to make some configuration changes to the SonarQube begin analysis task. 

- __SonarQube Endpoint__: The SonarQube url needs to be defined as a service in the administration console. From the SonarQube begin analysis task, from under the SonarQube server section click on 'Manage'. This will navigate you to the services tab in the administration console. 
    
   Click on the + New Service Endpoint dropdown and select Generic. This will open up the Generic connection window.  
   ![VSTS Connection Settings for SonarQube](/images/screenshots/tarun/SonarQube/VSTS_GenericServiceEndpoint.png)
   
   Enter the connection details of the SonarQube server and click OK.    
   ![VSTS Add SonarQube as a Generic Connection](/images/screenshots/tarun/SonarQube/VSTS_AddSonarQubeAsAnEndPoint.png) 
    
- __SonarQube ProjectKey__: Name this with respect to the solution you are performing an analysis on. I am setting this to Fabrikam.CI
- __SonarQube_ProjectName__: Name this to reflect the project or product being analysed. I am setting this to Fabrikam.WebServices
- __SonarQube_ProjectVersion__: By default this value is hard coded to 1.0. This isn't great because on every build SonarQube would overwrite the previous run results. Change the value of this field to the build number so you can see the correlation between sonarQube analysis results for the specific CI build number. I am using the predefined variable `$(Build.BuildNumber)`.
- __SonarQube_DatabaseSettings__: Nothing needs to be specified here. If you are using an earlier version of SonarQube < 5.2 then this applies to you. Since we are using SQL Windows Auth no configuration needs to be specified here. 
_ __Advanced__: Skip the advanced section for now. I'll cover Quality Gates in a future blog post. 

   ![VSTS SonarQube Begin Analysis Task Configuration](/images/screenshots/tarun/SonarQube/VSTS_SonarQubeBeginAnalysisConfiguration.png)

With that the configuration for SonarQube begin analysis task is complete. No configuration is required for the SonarQube end analysis task. The end analysis task is used to hand over the results file to SonarQube to process for analysing the results. Save the changes to build definition and queue a new build. 

# Trigger the CI Build to invoke code analysis using SonarQube 
Queue a new build for this build defintion. The build will be executed on the VSTS Agent. The Agent will use the configuration details in the SonarQube begin analysis task to connect to SonarQube endpoint and retrive the quality profile (or create one if it doesn't exist). 

![VSTS SonarQube Begin Analysis Task Configuration](/images/screenshots/tarun/SonarQube/VSTS_SonarQubeBeginAnalysisTaskDebug.png)

At the end of the analysis a file is handed over to SonarQube to processess. 

![VSTS SonarQube Begin Analysis Task Configuration](/images/screenshots/tarun/SonarQube/VSTS_SonarQubeEndAnalysisDebug.png)

Once the build is complete, you'll be able to see the SonarQube analysis summary in the build results section. 

![VSTS SonarQube Begin Analysis Task Configuration](/images/screenshots/tarun/SonarQube/VSTS_SonarQube_BuildSummaryMarkDown.png)

Clicking on Analyis results will navigate you into the specific Project in SonarQube to show you more details of the analysis...

![VSTS SonarQube Begin Analysis Task Configuration](/images/screenshots/tarun/SonarQube/SonarQube_VSTS_TechnicalDebtAnalysis.png)

# Mapping SonarQube Version to VSTS Build Number
By changing the Project Version configuration to use `$(Build.BuildNumber)` we now have a direct linking between the build number and the analysis result in SonarQube. 

![VSTS SonarQube Begin Analysis Task Configuration](/images/screenshots/tarun/SonarQube/IntegrateBuildNumberBetweenSonarQubeAndVSTSBuild.png)

# Summary
In this blogpost we walked through the configuration steps to trigger SonarQube code analysis from Team Build in VSTS. In doing so we also learned how to add Services as an endpoint in VSTS. In the next blogpost we'll learn how to fail a build in VSTS based on QualityGate configuration defined in SonarQube. I would also recommend you check out the following other blogposts on SonarQube...

- [Why care about DevOps?](http://www.visualstudiogeeks.com/blog/devops/marry-cloud-and-devops-enterprise-devops-is-for-real)
- [What is Technical Debt & why is it a problem?](http://www.visualstudiogeeks.com/blog/sonarqube/devops/Configure-TFS2015-with-SonarQube-using-BuildTask-to-Track-Technical-Debt)
- [How to install SonarQube as a Windows Service with SQL (Windows Auth) for code analysis with VSTS](http://www.visualstudiogeeks.com/blog/DevOps/Install-SonarQube-As-WindowsService-With-SQLServer-WindowsAuth-VSTS-TeamBuild)
 
 
 How are you tackling technical debt? Is SonarQube working well for you? 
 
 Tarun 