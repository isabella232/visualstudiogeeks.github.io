---
layout: post          #important: don't change this
title: "TFS 2015 Machine Groups - Walkthrough"
date: 2016-02-26 19:00:00 
author: Tarun Arora
categories:
- blog                #important: leave this here
- TeamFoundationServer
img:        #place image (850x450) with this name in /assets/img/blog/
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /assets/img/blog/thumbs/
keywords: "TFS2015, Labs, ContinuousDelivery, Testing, MachineGroups, VSTS, TFBuild"
---
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-56c6503fb913a4a1"></script>
Machine Groups is a new concept in TFS2015, you will see the option to add a machine group when configuring a build or release definition... Wondering what it is? In this blogpost, you'll learn what's machine group and how to configure and set up a MachineGroup. This blog post applies to both TFS and VSTS.
<!--more-->

## What is a Machine Group?
---
Simply put, Machine Group is a logical grouping of machines. The Machine Group holds metadata, connectivity, and login details of the machines in the group. Machine Group can directly be referenced from build and release denitions. 

<br/>

## Scenario
---
The Fabrikam Team has a lab environment in the Fabrikam.lab domain. Fabrikam.lab comprises of servers that serve different roles. The Fabrikam Team wants the ability to directly reference these machines from the build definition and release definition to deploy test agents on all the machines and trigger a distributed test run. Fabrikam.Lab is managed by the Fabrikam Environments Team who cannot share environment credentials with the Fabrikam Team. 

> In this blogpost, we'll walk through the process followed by the Fabrikam Environments Team to set up and configure the Machine Group Fabrikam-QA for the Fabrikam Team.

<img src="/assets/img/blog/tarun/TFSMachineGroup-FabrikamLabEnvironment.png" alt="TFSMachineGroup Fabrikam Lab Environment" style="width:100%;height:100%"><sub><center><b>Image 1 - TFS MachineGroup Fabrikam Lab Environment</b></center></sub>

The Machine Group will be accessed by a remote host; the remote host will be playing the role of a build agent or release agent. As illustrated in the following figure, the remote host is in the same network as the Machine Group and has a trust relationship with the Machine Group

<img src="/assets/img/blog/tarun/TFSMachineGroup-BuildAgentNetworkSetup.png" alt="TFSMachineGroup Agent setup" style="width:100%;height:100%"><sub><center><b>Image 2 - TFS MachineGroup Build Agent Setup</b></center></sub>


## Pre-requisities
----
The build agent uses Windows PowerShell remoting that requires the Windows Remote Management (WinRM) protocol to connect to the machines in the Machine Groups. WinRM needs to be enabled on a machine as a prerequisite before it can be added into the Machine Group. In this case, Kerberos will be used as the mode of authentication since the agent and Machine Group are in the same corp network.

<br/>

| Target Machine state | Target Machine trust with automation agent | Machine Identity | Auth Account | Auth Mode | Auth Account permission on target machine | Conn Type|
| ------------- | ------------- |----|---|---|---|---|
| Domain- joined machine in the corp network | Trusted  | DNS name | Domain account | Kerberos | Machine admin | WinRM HTTP | 

<br/>

## Preparing machines for Machine Group Setup
---
In the next few steps, we'll walk through how to configure WinRM on a machine, and you'll learn how to test connectivity through WinRM:

1. PowerShell 2.0 and Windows Management Framework 4.0 [Download](http://bit.ly/1kNlxuW "PowerShell Download link") are required to be installed on both the agent and machines in the Machine Group.

2. Log into the QA-Web1.Farbikam.lab machine, start Windows PowerShell as an administrator by right-clicking on the Windows PowerShell shortcut and selecting Run as Administrator.

3. By default, the WinRM service is configured for manual startup and stopped. Executing the winrmquickconfig -q command performs a series of actions:
    * Starts the WinRM service.
    * Sets the startup type on the WinRM service to Automatic.
    * Creates a listener to accept requests on any IP address.
    * Enables a firewall exception for WS-Management communications.
    * Registers the Microsoft.PowerShell and Microsoft.PowerShell. Workflow session configurations, if they are not already registered.
    * Registers the Microsoft.PowerShell32 session configuration on 64-bit computers, if it is not already registered.
    * Enables all session configurations.
    * Changes the security descriptor of all session configurations to allow remote access.
    * Restarts the WinRM service to make the preceding changes effective.

> The next few commands will prepare WinRM for Kerberos authentication.

+ Increase the maximum memory allocation per session:

``` console

    winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="300"}'

```

+ Next, increase the session timeout period:

``` console   

    winrm set winrm/config '@{MaxTimeoutms="1800000"}'

```

+ Allow the traffic between agent and Machine Group to be unencrypted:

``` console   

    winrm set winrm/config/service '@{AllowUnencrypted="true"}'

```

+ Disable basic authentication:

``` console

    winrm set winrm/config/service/auth '@{Basic="false"}'

```

+ Setup a  rewall exception to allow inbound traffic on port 5985; this is the default port used by WinRM when using HTTP:

``` console

        netshadvfirewall firewall set rule name="Windows Remote Management
        (HTTP-In)" profile=public
        protocol=tcplocalport=5985 remoteip=localsubnet new remoteip=any

```
    
+ Disable digest for client authentication:

``` console

   winrm set winrm/config/client/auth '@{Digest="false"}'

```

+ Set service authentication to use Kerberos:

``` console

   winrm set winrm/config/service/auth '@{Kerberos="true"}'

``` 

+ Trust all connections between agent and Machine Group:

``` console

   winrm set winrm/config/client '@{TrustedHosts="*"}'
   Set-Item WSMan:\localhost\Client\TrustedHosts *

```

+ Restart the win-rm service: Restart-Service winrm–Force
+ To ensure Kerberos authentication is enabled on WinRM, run the following command:

``` console

    winrm get winrm/config/service/auth

```

<br/>

<img src="/assets/img/blog/tarun/TFSMachineGroup-KerberosAuthentication.png" alt="TFSMachineGroup Kerberos Authentication" style="width:100%;height:100%"><sub><center><b>Image 3 - TFS MachineGroup Kerberos Authentication</b></center></sub>

+ Now, let's validate whether WinRM has correctly been set up on QA-Web1. Fabrikam.lab. Log into another VM in the lab, in this case QA-Web2.Fabrikam. lab. Launch PowerShell as an administrator by right-clicking on the Windows PowerShell shortcut and selecting Run as administrator. Execute the following command:
       
``` console

    Test-Wsman –computerName QA-Web1.Fabrikam.lab

```

<img src="/assets/img/blog/tarun/TFSMachineGroup-TestWsman.png" alt="TFSMachineGroup Test Wsman" style="width:100%;height:100%"><sub><center><b>Image 4 - TFS MachineGroup Test Wsman</b></center></sub>

+ Execute the following command to check the port WinRM is listing on:

``` console

    winrm e winrm/config/listener

```

<img src="/assets/img/blog/tarun/TFSMachineGroup-TestListener.png" alt="TFSMachineGroup Test Listener" style="width:100%;height:100%"><sub><center><b>Image 5 - TFS MachineGroup Test Listener</b></center></sub>

> Execute the following command should you want to change the port WinRM is currently configured to listen on: 

``` console

    Set-Item WSMan:\localhost\listener\*\Port 8888

```

+ Most importantly, validate that you are able to invoke the PSSession on QA-Web1. Fabrikam.lab by manually running the following command from QA-Web2. Fabrikam.lab. Once you execute the  rst statement, you'll receive a prompt to enter your credentials. Enter your domain account that has admin permissions:

``` console

    $cred = get-credential

```

+ Executing the next command will use your domain account to connect to the destination server; DNS will be used to resolve the destination name:

``` console

    Enter-Pssession -ComputerNameQA-Web1.Fabrikam.lab–Credential $cred

```
 
<img src="/assets/img/blog/tarun/TFSMachineGroup-TestDestinationConnectivity.png" alt="TFSMachineGroup Test Connectivity" style="width:100%;height:100%"><sub><center><b>Image 6 - TFS MachineGroup Test Connectivity</b></center></sub>


+ Follow steps 1 to 5 to configure WinRM on other machines in the lab. Follow step 6 to validate WinRM connectivity before moving forward.

<br/>

## How to configure MachineGroup in TFS (VSTS)
---
+ Navigating to the test hub in the Fabrikam Team Web Portal, on the Machines page, click on the + icon to create a new Machine Group:

<img src="/assets/img/blog/tarun/TFSMachineGroup-Configure01.png" alt="TFSMachineGroup Configuring in TFS VSTS" style="width:50%;height:100%"><sub><center><b>Image 7 - TFS MachineGroup Configuring in TFS (VSTS)</b></center></sub>

+ Enter the details as illustrated in the following screenshot:

<img src="/assets/img/blog/tarun/TFSMachineGroup-ConfigureMachineGroup02.png" alt="TFSMachineGroup Configuring" style="width:100%;height:100%"><sub><center><b>Image 8 - TFS MachineGroup Configuring in TFS (VSTS)</b></center></sub>

+ The WinRM protocol in Fabrikam.lab will use HTTP since the remote machine has a trust relationship with Fabrikam.lab.Add and the details for all the machines. Now, click on Done to complete the setup:

<img src="/assets/img/blog/tarun/TFSMachineGroup-Configure03.png" alt="TFSMachineGroup Configuring" style="width:100%;height:100%"><sub><center><b>Image 9 - TFS MachineGroup Configuring in TFS (VSTS)</b></center></sub>

<br/>

## How it works
---
The Fabrikam-QA Machine Group setup uses a common administrator credentials for all machines in the Machine Group. It is alternatively possible to specify different credentials for the individual machines added in the Machine Group:

+ To enter credentials per machine, check the option Use custom credentials for each machine along with global credentials.

<img src="/assets/img/blog/tarun/TFSMachineGroup-Result01.png" alt="TFSMachineGroup Results" style="width:100%;height:100%"><sub><center><b>Image 9 - TFS MachineGroup Results in TFS (VSTS)</b></center></sub>

+ The password  field is masked in the user interface. In addition to this, the value of this  field is not printed in any of the log  files either.
+ The tags provide a great way to query for machines with in the Machine Group. For example, when using the test agent deployment task in build de nition, you can specify a Machine Group and use Tags to  lter the execution of the action on machines that include the Tag.
+ Machine Groups, at the moment, support limited scenarios mainly domain joined on premise machine and standalone machines in Azure. Refer to http://bit.ly/1NFqYma for a full list of supported scenarios.

<br/>


## Learn how to take advantage of Test Automation features in TFS 2015
---
In the next blog post we'll learn how to use machine groups in build definition and release definitions to deploy on machine groups. 

Itching to get started with the all new web-based test automation features in TFS 2015....?

The Team Foundation Server 2015 Cookbook includes over 80 hands-on DevOps and ALM focused labs for Scrum Teams to enable software teams to champion the implementation of modern application lifecycle and DevOps tooling using Team Foundation Server 2015. Among other things the chapter on release management includes hands on labs for...

<br/>

*   Running NUnit tests in the CI Pipeline using TFBuild
*	Creating and Setting up a Machine Group
*	Deploying Test Agent through TFBuild Task
*	Distributed Test Execution on a Lab Machine Group
*	Triggering Selenium Web Tests on a Selenium Test Grid using TFBuild
*	Integrating Cloud Load Testing Service in TFBuild
*	Analyzing Test Execution results in Test Runs view
*	Exporting and Importing Test Cases in Excel from TFS
*	Copying and Cloning Test Suites and Test Cases 
*	Exporting Test Artifacts and Test Results from Test Hub
*	Charting Testing Status on Dashboards in Team Portal

 
### Team Foundation Server 2015 Cookbook...
---
To order a copy of the Team Foundation Server 2015 Cookbook...

#### Order here - UK 
---
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//ws-eu.amazon-adsystem.com/widgets/q?ServiceVersion=20070822&OneJS=1&Operation=GetAdHtml&MarketPlace=GB&source=ac&ref=tf_til&ad_type=product_link&tracking_id=tararo-21&marketplace=amazon&region=GB&placement=1784391050&asins=1784391050&linkId=&show_border=true&link_opens_in_new_window=true"></iframe>

#### Order here - USA 
---
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//ws-na.amazon-adsystem.com/widgets/q?ServiceVersion=20070822&OneJS=1&Operation=GetAdHtml&MarketPlace=US&source=ss&ref=ss_til&ad_type=product_link&tracking_id=tararo-20&marketplace=amazon&region=US&placement=B0148S9GUE&asins=B0148S9GUE&linkId=WFCNWAEUMMGD3Q4T&show_border=true&link_opens_in_new_window=true"></iframe>

<br/>
Team Foundation Server 2015 Cookbook "Over 80 hands-on DevOps and ALM-focused recipes for Scrum Teams to enable the Continuous Delivery of high-quality Software... Faster!"

##### About This Book
* Release high quality, reliable software quickly through building, testing, and deployment automation
* Improve the predictability, reliability, and availability of TFS in your organization by scheduling administration and maintenance activities
* Extend, customize, and integrate tools with TFS, enabling your teams to manage their application lifecycles effectively

##### Who This Book Is For
This book is aimed at software professionals including Developers, Testers, Architects, Configuration Analysts, and Release Managers who want to understand the capabilities of TFS to deliver better quality software faster. A working setup of TFS 2015 and some familiarity with the concepts of software life cycle management is assumed.

### What You Will Learn
* Creating a Team Project with Dashboards, Assigning License, Adding users, and Auditing Access
* Setting up a Git repository in an existing TFVC-based Team Project
* Setting up branch policies and conducting Pull requests with code reviews
* Mapping, assigning and tracking work items shared by multiple teams
* Setting up and customizing Backlogs, Kanban board, Sprint Taskboard, and dashboards
* Creating a Continuous Integration, Continuous Build, and Release Pipeline
* Integrating SonarQube with TFBuild to manage Technical Debt
* Triggering Selenium Web Tests on a Selenium Test Grid using TFBuild
* Using Visual Studio Team Services Cloud load testing capability with new Build framework
* Extending and customizing the capabilities of Team Foundation Server using API and Process Editor

---

Hope you enjoy reading the book... 
