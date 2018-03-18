---
layout: post
title: "How to organzie VSTS build output into seperate folders?"
date: 2018-03-17
author: tarun
tags: ["DevOps", "Build"]
categories:
- "DevOps"
- "Build"
img: "/images/screenshots/tarun/mar18/006_organizeBuildOutputIntoFolders.jpg"
description: "Wondering how to organize your VSTS Build Output into individual folders so you can directly consume it in your release pipeline for the purposes of deployment? In this blogpost on DevOps I'll show you how to use the copy task in VSTS Build Pipeline to structure the build artifact such that the output is structured in multiple folders ready to be consumed."
permalink: /DevOps/HowToOrganizeBuildOutputIntoSeperateFoldersUsingVstsBuild
published: true
keywords: "devops, develop, continuous delivery, continuous integration, devops wiki, continuous deployment, devops tutorial, ci server, devops model, devops definition, continuous integration tools, cloud devops, devops principles, agile and devops, agile devops, devops delivery model, devops emphasizes on, devops duties, devops setup, devops ideas, ci cd pipeline, codepipeline, ci pipeline, ci, continuous integration tools, continuous development, integration tools, continuous, ci server, build server, how to implement continuous integration, build and deployment automation, Azure, VSTS, TFS, alm, Visual Studio, VisualStudio, microsoft team build"
---
I have seen teams not care about the structure of the build artifact, instead add several steps in their release pipeline to unclutter the binaries generated from the build output. It is fairly trivial to control the structure in which the build output gets published as an artifact to your TFS or VSTS build. In the spirit of pushing left (DevOps way of working), in this blogpost I'll show you how to easily structure the build output in seperate folders by using the copy task in the build pipeline.
<!--more-->

# Problem Statement 
I have a solution file which when processed through the build pipleine generates a build output and add's every folder that has any dll's into the build output. Let's double click to see where the real problem is,
+ Folders that I have no interest in have been added as a build artifact

    ![Deafult buildouput add's all folders that produce dll's in bin directory](../images/screenshots/tarun/mar18/002_defaultoutputhasfoldersthatidontneed.jpg) 

+ The folders are nested into sub folders, and sub folders and further sub folders for stuff that I am interested in

    ![Deafult buildouput nested 3 levels in](../images/screenshots/tarun/mar18/001_defaultbuildoutput.jpg)
  


# Solution

A vaniella build pipeline add's one `copy` step and one `publish` step in the build pipeline...

![One copy and one publish step in default build pipeline](../images/screenshots/tarun/mar18/003_defaultpipelineonecopyandonepublishstep.jpg)

The build generates the binaries in the `$(build.defaultworkingdirectory)` folder, the copy step simply copies all the binaries from the default working directory `$(build.defaultworkingdirectory)` using the format `**\bin\$(BuildConfiguration)\**` into the artifact staging directory `$(build.artifactstagingdirectory)`... The publish step as you would expect published everything that it finds in the `$(build.artifactstagingdirectory)` 

> The build copy step is your friend if you use it correctly...

Instead of overloading just one copy step to copy everything, I have added multipe copy steps. I've qualified each step to full qualify the folders I need to get rid of the multi level hierarchies in the artifact folder, see the example below... 

``` PowerShell
# Fully qualify the location where the binaries are
# this will remove the sub folders from the build output

$(system.defaultworkingdirectory)\xxx\MessageEngine\bin\$(BuildConfiguration)\

# Specify a target folder name where you want the binaries copied
$(build.artifactstagingdirectory)\ME.Service

```

![One copy and one publish step in default build pipeline](../images/screenshots/tarun/mar18/004_multicopystepfullyqualifyandaddsubfolder.jpg)


With the new copy tasks in, the build output now looks like this...

![One copy and one publish step in default build pipeline](../images/screenshots/tarun/mar18/005_buildoutputafterproposedmultistepcopy.jpg)

I hope you find this useful... #DevOpsOn...

Know a better way of doing this? leave a comment...

Tarun 