---
layout: post
title: "Team Services - Use one build definition to build all branches & release selectively"
date: 2017-09-27
author: tarun
tags: ["DevOps", "BuildPipeline"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/Sep17/OneBuildToRuleThemAll.png"
description: "A trick to leverage one build definition in Team Services to build all branches. How to update the build name from the build pipeline to reflect the name of the branch or the quality of the artifacts. One Build to Rule them all! How to map branches to release environments in Team Services."
permalink: /DevOps/TeamServicesOneBuildDefinitionToRuleThemAll
published: true
keywords: "DevOps, VSTS, BuildOnce, DeployEverywhere, TeamServices, One build for all branches, One build definition, BuildDefinition, TFS Build, Team Build, Build Pipeline, Release Pipeline, Continuous Delivery with Team Services"
---
With the flexibility in branching, it's very easy to create a branch per feature. Sometimes I notice that people lack the same enthusiasm when it comes to creating build pipelines for these new branches. The downside of course is the more build definitions you create, the more administration overhead in keeping them all up to date as you make changes to your build pipeline. **Wouldn't it be great if you could just use one build definition to build all branches?**
<!--more-->
In this blogpost I'll walk you through a trick to use just one build definition to build all branches in your repository... 

> One build to rule them all!  

![](/images/screenshots/tarun/Sep17/HappySurprised.gif)


For the purposes of this walkthrough let's assume you have the following branches

- *master* main line for production
- *develop* integration for all features 
- *feature/myfeature1* feature development branch

Let's see first how you can set up one build definition to trigger for all of these branches... 
+ Let's start off by creating a new build, choose an empty process
    ![image.png](/images/screenshots/tarun/Sep17/image-b6175ef2-9bfb-48de-81c3-1a45f6f9b927.png)
+ Call the build `OneBuildForAllBranches` and select a build pipeline, now navigate to the triggers section in the build and set the `Trigger` as `Continuous Integration`. Click the branch filter drop down and in the search box type `*` and press enter
  ![image.png](/images/screenshots/tarun/Sep17/image-4af4d89c-fb8c-4237-97d2-8430422dd66a.png)

> The build definition is now set up to trigger itself for changes in any branch in your repository. You could optionally reduce the filter scope by using specifying the branch filters to be excluded from the `*` all branch setup. 

+ To test this is working, you can make changes in any of these branches and you'll see the build executes just fine... 
  ![image.png](/images/screenshots/tarun/Sep17/image-3429f0df-7f1a-41ea-a575-8b4b6265b74b.png)

> O wait! That's a problem, I want to now identify the build by the branch it was executed for, otherwise I might accidently deploy my feature branch into production directly... 

## How to reflect branch quality in build name?

+ In your build pipeline add a `PowerShell` task and copy the below shared code snippet...

``` PowerShell
write-host $env:BUILD_SOURCEBRANCHNAME

if ($env:BUILD_SOURCEBRANCHNAME -eq "Develop"){
     Write-Output ("##vso[build.updatebuildnumber]" + $env:BUILD_BUILDNUMBER+"-beta")
      write-host "setting version as -beta"
}
else 
{
      if ($env:BUILD_SOURCEBRANCHNAME -ne "master"){
           Write-Output ("##vso[build.updatebuildnumber]" + $env:BUILD_BUILDNUMBER+"-alpha")
           write-host "setting version as -alpha"
       }
}
```

+ In the PowerShell script above, I am simply using the predefined variables and prefixing the name of the build `beta` or `alpha` depending on which branch the build is being created from. Now if you run the builds for all the branches you'll see the quality reflected in it's name... 
   ![image.png](/images/screenshots/tarun/Sep17/image-0958ed3e-b1e5-4a7f-867e-303d2b71d653.png)

<hr/>

> Ok cool! Now you are probably thinking, how can I stop my alpha build from accidently rolling out into the production environment?

## How to filter branchs in environments in release pipeline?

+ Start off by creating a release pipeline and link it to your build pipeline. Create three environments, for example `DevTest`, `PreProd` and `Prod`
   ![image.png](/images/screenshots/tarun/Sep17/image-857253b2-8c08-495a-b396-9d29ec920b30.png)

+ For the pre-prod environment, click on the `Pre-deployment Condition` icon and select `Artifact filters`. This setting would block the `Feature\*` branches from being released into the Pre-Prod environment. 
   ![image.png](/images/screenshots/tarun/Sep17/image-0b022ddf-1ee9-43a4-b255-9f385b09ca84.png)

+ For the Prod environment, click on the `Pre-deployment Condition` icon and select `Artifact filters`. This setting would block all branches except `master` from being released into the Prod environment. 
   ![image.png](/images/screenshots/tarun/Sep17/image-03d0e0af-efff-4b32-8894-5a05dc996e49.png)

## Summary 
In this blogpost we saw how easy it is customize the build name by using the name of the branch it was triggered from and then how to set up the release pipeline to filter out specific artifacts from being released into specific environments. Hope you found this useful... 

Tarun 
