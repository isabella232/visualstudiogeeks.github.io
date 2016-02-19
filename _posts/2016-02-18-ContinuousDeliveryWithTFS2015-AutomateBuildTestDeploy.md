---
layout: post          #important: don't change this
title: "Continuous Delivery with Team Foundation Server 2015"
date: 2016-02-18 22:37:00 
author: Tarun Arora
categories:
- blog                #important: leave this here
- TeamFoundationServer
img:        #place image (850x450) with this name in /assets/img/blog/
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /assets/img/blog/thumbs/
keywords: "TFS2015, ContinuousDelivery, DevOps, ALM, ReleaseManagement"
---
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-56c6503fb913a4a1"></script>
Continuous delivery is a software engineering approach in which teams produce software in short cycles, ensuring that the software can be reliably released at any time... What emotions does the word "release" trigger in you? Relief? Elation? A fist-pumping sense of accomplishment? If your teams are still 'testing and deploying' **manually**, releases probably don't excite you... In this blog post we'll learn how Team Foundation 2015 enables you to release high quality, reliable software quickly through release automation... 
<!--more-->

### Making a case for Continuous Delivery...  
---

Software delivery has gone through a revolution in the last decade. The introduction of Agile practices and lean frameworks, such as Scrum, Kanban, XP, and RUP, among others, have demonstrated that iterative feedback-driven development helps to cope with changes in the marketplace, business, and user requirements. Lean processes also help minimize waste and maximize value delivery to end users. Better DevOps practices encouraging Continuous Integration, continuous deployment, continuous delivery, and continuous feedback along with better tooling are enabling organizations to break the silos between teams. 

> Continuous Delivery isn't just for Unicorns! If you are working in professional software development you can and should strive for continuous delivery...   

The foundation to Continuous Delivery is a well established and automated release process. 

> Manual Software Deployment != Continuous Delivery 

<br/>

### Why Microsoft Visual Studio Team Foundation Server 2015?
---

<img src="/assets/img/blog/tarun/ContinuousDeliveryWithTFS2015-VSFamily01.png" alt="Visual Studio Product Family" style="width:100%;height:100%"><sub><center><b>Image 1 - Microsoft Visual Studio Product Family</b></center></sub>

The Visual Studio family of tools and services now enables heterogeneous software development across various platforms. The experience of using open source tooling within the product has improved tremendously. Open source solutions are being given first class citizen status, and more of these solutions are being pre-packaged into the product. This gives us a clear indication that Microsoft wants to become the platform of choice for every developer, independent of the technology or platform. There is a huge overlap between the tools and services within the Visual Studio family of tools. Microsoft Visual Studio Team Foundation Server 2015 is at the center of Microsoft's ALM solution, providing core services such as version control, work item tracking, reporting, automated - testing, builds and releases. TFS helps organizations communicate and collaborate more effectively throughout the process of designing, building, testing, and deploying software, ultimately leading to increased productivity and team output, improved quality, and greater visibility of an application's life cycle.

### Microsoft Team Foundation Server 2015 - Web-based Release Management
---

On June 3rd, 2013 Microsoft acquired the InRelease product from InCycle Software. The InRelease product was re-branded as Microsoft Release Management and integrated into Team Foundation Server 2013. Microsoft Release Manager gave Microsoft a position in the growing release management market. While Release Manager shipped along with Team Foundation Server, it required separate installation and setup. Though various improvements were made to improve the integration between the two products, they still felt disjointed at several places. The WPF-based desktop client was clunky and limiting. Release Manager did not support non .NET applications and could not be used on non-Windows platforms. It was clear that Release Manager was only a stop-gap solution and would need to be replaced by a proper solution.
<br/>
The old release manager solution is being replaced by an all new web-based release management solution. The new web-based Release Management solution has debut in TFS 2015 Update 2. It is very well integrated into the product. No separate installation or configuration is required to start using the new Release Management solution. The security infrastructure of Release Manager is different from the previous version in that it does not manage its own groups and permissions. New permissions are introduced in TFS for Release Management, such as "Create release definitions", "Create releases", and "Manage approvers". Default values for these permissions are set for specific groups at the Team Project level. These permissions can then be overridden for the groups or individual users, for a specific release definition or for a specific environment within a release definition.

<img src="/assets/img/blog/tarun/ContinuousDeliveryWithTFS2015-ReleaseAgentArchitecture.png" alt="TFS Release Agent Architecture" style="width:100%;height:100%"><sub><center><b>Image 2 - Release Agent Architecture</b></center></sub>

Both Team Build and Release Management share the same agent—pool and queue infrastructure. Unified agent infrastructure reduces administration and set up overhead. The tasks used to orchestrate the actions are also shared between Build and Release. This significantly reduces the learning curve for release management. The new solution is web based, open, extensible and fully cross-platform. The underlying framework between Team Build and Release Management is the same, so you get the same real-time console output in Release Management as you get with Team Build. The Release Definitions supports change revision and diff functionality similar to that in Build Definitions. Release Management supports Draft Releases similar to the Draft Build functionality in Team Build. 

> With so much common between Build and Release Management, so much so that Build also has access to deployment tasks, you may ask how are the build and release management different? 

While the line between Build and Release Management is blur because both share so much in common. **The key difference is that Deployment is just one of the activities performed in Release Management.** As illustrated in the image below, the new Release Management solution allows creating release pipelines. A release pipeline can consist of one or more environments. Each environment can have one or more physical or virtual deployment targets. Environments provide pre-release and post-release approval workflow as well as tasks for testing and deployment.
<br/>
<img src="/assets/img/blog/tarun/ContinuousDeliveryWithTFS2015-TFSReleasePipeline.png" alt="TFS Release Pipeline" style="width:100%;height:100%"><sub><center><b>Image 3 - Release Pipeline</b></center></sub>
<br/>

### Learn how to take advantage of Release Management in TFS 2015
---

Itching to get started with the all new web-based Release Management solution in TFS 2015....?

The Team Foundation Server 2015 Cookbook includes over 80 hands-on DevOps and ALM focused labs for Scrum Teams to enable software teams to champion the implementation of modern application lifecycle and DevOps tooling using Team Foundation Server 2015. Among other things the chapter on release management includes hands on labs for...

<br/>

* Creating a release definition in Team Web Portal
* Mapping Artifacts to a release definition
* Configuring a release definition for Continuous Deployment
* Adding and Configuring Environments in a release definition
* Configuring Security for release definitions
* Configuring Global and Local Variables for a release
* Deploying an Azure Website using release management
* Deploying IIS Web Application using release management
* Tracking a release in release management
 
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
