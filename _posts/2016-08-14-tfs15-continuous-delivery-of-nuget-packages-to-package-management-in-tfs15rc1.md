---
layout: post
title: "Continuous delivery of Nuget packages to Package Management in TFS15 RC1"
date: 2016-08-14
author: utkarsh 
tags: ["TFS15", "DevOps", "TeamFoundationServer"]
categories:
categories:
- "TFS15"
- "DevOps"
- "TeamFoundationServer"
img: "/images/screenshots/utkarsh/tfs15-package-mgmt/package-mgmt.png"
description: "Continuous delivery of Nuget packages to Package Management in TFS 15 RC1"
permalink:  /DevOps/continuous-delivery-of-nuget-packages-to-package-management-in-tfs15rc1
keywords: "TFS15"
---

TFS 15 RC1 got [released](https://blogs.msdn.microsoft.com/bharry/2016/08/08/tfs-15-rc1-is-available/) couple of days ago and one of the most awaited feature from on-prem customers was Package Management. 

In this post we will see how we can publish our common code as Nuget packages so that it can be shared across the organization. Also, we will see how we can keep these packages up to date by publishing them to our TFS15 package management server as part of our continuous delivery process.

<!--more-->

![Package Management](/images/screenshots/utkarsh/tfs15-package-mgmt/package-mgmt.png)

## Install Package Management extension ##

To use the TFS15 Package Management on on-prem, you first need to install the Package Management extension. The package management extension now comes as part of TFS15 RC1 installation. To install, you first need to go to your local marketplace.

On your on-prem server home (`http://localhost:8080/tfs` for example) click on marketplace icon and then click `Browse TFS extensions`

![InstallExtension1](/images/screenshots/utkarsh/tfs15-package-mgmt/install-ext-1.png)

You will be taken to your local extension marketplace.

![InstallExtension2](/images/screenshots/utkarsh/tfs15-package-mgmt/install-ext-2.png)

Click on `Package Management` extension and install it on to your desired  TFS collection.

![InstallExtension3](/images/screenshots/utkarsh/tfs15-package-mgmt/install-ext-3.png)

## Create a package feed in Package hub ##

> A package feed is a container of your packages. Package clients similar to `Nuget Package Manager` of Visual Studio will be able to subscribe to this feed and receive packages. 

Once you install the extension, you should see `Package Feeds` in your navigation bar under `Build/Release` menu. Click `Package Feeds`

![GoToPackageHub](/images/screenshots/utkarsh/tfs15-package-mgmt/goto-package.png)

You will be navigated to `Package Feeds` screen.

![PackageTab](/images/screenshots/utkarsh/tfs15-package-mgmt/package-tab.png)

Click `New Feed`. This opens a new dialog. Provide the feed details.

![FeedDetails](/images/screenshots/utkarsh/tfs15-package-mgmt/new-feed.png)

Once you click OK, a new Nuget feed will be created.

![FeedDetailsCreated](/images/screenshots/utkarsh/tfs15-package-mgmt/new-feed-created.png)

Note down the feed URL from the feed details page on the right side. We will use it later in the post to publish our packages to this feed. Also, If you want to use your own instance of `nuget.exe` you can download it from here.

## Manage feed Security ##

You can right click on the feed, click `Edit` - to manage its security, such as who can contribute packages, who can consume the packages etc.

![FeedPermissions](/images/screenshots/utkarsh/tfs15-package-mgmt/feed-permissions.png)


## Build and publish Nuget packages solution ##

Now we are in a position to build our solution and publish the nuget packages. For this post, I have two of my commonly used libraries which I would like to publish as Nuget packages.

My code structure is as below.

![CodeCheckin](/images/screenshots/utkarsh/tfs15-package-mgmt/code-checkin.png)

Now to build the solution, I have created a simple build definition with following steps.

![BuildSteps](/images/screenshots/utkarsh/tfs15-package-mgmt/build-steps.png)

Most of the steps are self explanatory. However, I would like to briefly describe the following important steps.

1. [Nuget Packager](https://www.visualstudio.com/en-us/docs/build/steps/package/nuget-packager) : I use this task to package my code in to nupkg files. This task scans either csproj/nuspec file and packages in to nupkg file.
2. In the next task I publish the generated nupkg file as artifacts of this build. This task also copies packages in to `$(build.artifactstagingdirectory)`
3. [Nuget Publisher](https://www.visualstudio.com/docs/build/steps/package/nuget-publisher) : Finally, I use this task to find all the packages in the `$(build.artifactstagingdirectory)` and publish this to our feed.

Let's see packaging and publishing task in detail.

### Nuget Packager ###
As the name indicates, the first task is is used to pack the code as Nuget package. The details of the task configuration is as below.

![Package](/images/screenshots/utkarsh/tfs15-package-mgmt/nuget-packager.png)

### Nuget Publisher ###
As the name indicates, this task is is used to publish the code to our Nuget feed. The details of the task configuration is as below.

![Publish](/images/screenshots/utkarsh/tfs15-package-mgmt/nuget-publisher.png)

**Notice** that for publishing to local package feed you need to select feed type as `Internal Nuget Feed`.

The next parameter is to provide the feed URL generated during the feed creation step above. For me it was as below.

`http://localhost:8080/tfs/DefaultCollection/_packaging/MyCompanyNuget/nuget/v3/index.json`

Save the build definition and Queue a new build. If everything is set right, your build should be green and you should see your packages available in the feed.

![Feed](/images/screenshots/utkarsh/tfs15-package-mgmt/feed.png)

## Consuming the Nuget feed ##

You or your team is now ready to consume these Nuget packages. The details page, when you create the feed, already gives instructions on how to add this feed as a source to your package manager client (for example Visual Studio's Nuget Package Manager). The command, to add this feed as a source is

```shell
nuget.exe sources Add -Name "MyCompanyNuget" -Source http://localhost:8080/tfs/DefaultCollection/_packaging/MyCompanyNuget/nuget/v3/index.json
``` 

Execute the above command on the client machine command prompt. This should update the Nuget.config file and you will see a new Nuget source.

![NugetConfig](/images/screenshots/utkarsh/tfs15-package-mgmt/nuget-config.png)

If you are Visual Studio user, you can also add this source by going to `Tools | Options | Nuget Package Manger | Package Sources` and providing feed URL as below.

![NugetConfig](/images/screenshots/utkarsh/tfs15-package-mgmt/vs-nuget-source.png)

## Summary ##

That's it. You have integrated publishing your Nuget Packages to the local package feed within your TFS 15. So every time you update your package, you are publishing the latest packages to the Nuget feed, thus making sure your team always consumes latest packages.

Thanks for reading.