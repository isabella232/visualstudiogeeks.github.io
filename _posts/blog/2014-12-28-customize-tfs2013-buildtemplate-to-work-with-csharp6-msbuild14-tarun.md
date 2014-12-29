---
layout: post          #important: don't change this
title: "How to compile .NET 2015 Preview projects with TFS 2013 Build"
date: 2014-12-28 23:45:00
author: Tarun Arora
categories:
- blog                #important: leave this here
- "TFS"
- "Build"
- 
img:        #place image (850x450) with this name in /assets/img/blog/
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /assets/img/blog/thumbs/
---
Struggling to compile your .NET 2015 Preview (.NET 4.6) solutions using TFS 2013 build? That's because TFS 2013 uses MSBuild 12.0 as the default version. Your Microsoft .NET 2015 Preview (.NET 4.6 / C# 6 beta) solutions with fail compilation if you use TFS 2013 Build agents. Let's see how this can be fixed...  
<!--more-->

If you like this post, don't forget to subscribe to new blog post alerts http://feeds.feedburner.com/visualstudiogeeks/otas 

### How to customize build template to point to MSBuild 14.0? ###

1. You need to amend the build templates to point to MS Build version 14.0. The easiest way to do this is to create a copy of the build template for backup.  
![TFS 2013 Build Templates](/assets/img/blog/tarun/post03_tfs2013buildtemplates.jpg)
2. Open the template in Visual Studio and search (Ctrl + F) for "ToolVersion"
3. Click F4 to load the properties window 
4. Change the MSBuild version to "14.0"
5. Search for other occurrences of ToolVersion and override the values for MSBuild only

![Customize TFS 2013 template to work with MSBuild 14.0](/assets/img/blog/tarun/post03_tfs2013buildtemplatecustomization.jpg)

	Note: Restart the build service to ensure that the updated version of the build template is picked up by the build agent. 

Queue a new build, your .NET 4.6 project should compile just fine.

Hope you found this post helpful. If you like this post, don't forget to subscribe to new blog post alerts http://feeds.feedburner.com/visualstudiogeeks/otas 
