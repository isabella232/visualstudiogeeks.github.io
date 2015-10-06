---
layout: post          #important: don't change this
title: "AppVeyor extension for Visual Studio 2013/2015"
date: 2015-10-06 21:16:00 
author: Utkarsh Shigihalli
categories:
- blog                #important: leave this here
- "extensibility"
- appveyor
- "visual studio"
- 
img:        #place image (850x450) with this name in /assets/img/blog/
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /assets/img/blog/thumbs/
---
[AppVeyor](http://www.appveyor.com/) is a well known cloud build server, which integrates with many source controls like VSO (Visual Studio Online), GitHub, BitBucket etc. Like many others do, we love Appveyor. However, most of the time we spend our time in Visual Studio and interacting with Appveyor was not possible from within Visual Studio. So we decided to build an extension!
<!--more-->


> [**Download**](https://visualstudiogallery.msdn.microsoft.com/54fd33fb-cd0e-4b1e-b113-a5ebb17fff20) from VS Gallery

## So what is in the first version? ##
- Seamless integration within Visual Studio
- Monitor Builds and build status
- Start/Stop builds
- Go to individual project specific pages from extension.

## Show me some screenshots! ##
Once you install the extension you will see a new menu item "AppVeyor" in Visual Studio's View menu.

![Alt text](/assets/img/blog/utkarsh/appveyor_view.jpg)

Clicking that will open a new "AppVeyor" toolwindow

![Alt text](/assets/img/blog/utkarsh/appveyor_toolwindow_empty.jpg)

Before you see your configured projects you need to enter your [AppVeyor API token](https://ci.appveyor.com/api-token) in Options. This is required for extension to communicate with your AppVeyor account. We use [bearer token authentication](http://www.appveyor.com/docs/api#authentication) to access AppVeyor API. 

You can do that, by clicking gears icon on the toolbar. Clicking that will open options window.

![Alt text](/assets/img/blog/utkarsh/appveyor_options.jpg)

Once you have entered correct API token for your account, extension will automatically fetch the projects and display it in the tool window. The tool window now will look like below. 

![Alt text](/assets/img/blog/utkarsh/appveyor_toolwindow_full_annotate.jpg)

As you can see in the screenshot above, the toolwindow looks similar to web interface of AppVeyor.

The additional buttons on the right hand side of the toolwindow will allow you to start and stop the build, and browse additional options which at this moment take you to web interface. **Our plan is to get these also within the extension in future releases**.

> 
**PLEASE NOTE:**

- This a **beta version** and our extension does only part of what AppVeyor supports. 
- Tested only against our GitHub repositories as we do not have enterprise account with AppVeyor :-)
- You may encounter few bugs in the extension, but we promise we did not leave them intentionally, so please report them to us and we will plan to fix. 

We hope you like it. If you do, please share and rate in the gallery.

<a href="http://ctt.ec/ycnlf"><img src="http://clicktotweet.com/img/tweet-graphic-4.png" alt="Tweet: Monitor #AppVeyor builds inside #VisualStudio - http://ctt.ec/ycnlf+ with this extension. Developed by @onlyutkarsh @arora_tarun" /></a>
