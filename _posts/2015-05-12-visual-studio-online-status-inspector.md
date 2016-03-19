---
layout: post          #important: don't change this
title: "Visual Studio Online Status Inspector"
date: 2015-05-12 05:15:00 
author: Utkarsh Shigihalli
categories:
- blog                #important: leave this here
- "Visual Studio Extensibility"

img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---

Occasionally when Visual Studio Online (VSO) is down or having some issues, we head over to the [status overview](https://www.visualstudio.com/en-us/support/support-overview-vs.aspx) page to monitor its status. How nice would it be to monitor the status while you code in Visual Studio? I thought it would be useful, and I just developed a tiny extension to monitor the status of VSO from Visual Studio itself. 
<!--more-->

The extension quitely sits in the Visual Studio status bar displaying an icon - based on whether Visual Studio Online is running smooth or has some issues or completely down. 

Here is a quick **walkthrough of its features**

Once you install the extension you will see a small icon in your Visual Studio status bar based on the status of the VSO.

![Alt text](/images/screenshots/utkarsh/vso_status_inspector_green.png "VSO is up")
![Alt text](/images/screenshots/utkarsh/vso_status_inspector_red.png "VSO is down")
![Alt text](/images/screenshots/utkarsh/vso_status_inspector_yellow.png "VSO is yellow")

As of now, you cannot perform any interaction with the icon displayed. 

Now, the extension by default polls for the status every 60 seconds. You can also see the details of the poll. To view that, Open the Output window and choose `VSO Status Inspector` from the dropdown. 

![Alt text](/images/screenshots/utkarsh/vso_status_inspector_output.png "VSO output")

(P.S: I had set the poll time to be 30 seconds)

Finally, if you ever want to change the poll time, you can do that easily too. Go to `Tools -> Options` and then search for `VSO Status Inspector`. Change the poll interval to whatever you like.

![Alt text](/images/screenshots/utkarsh/vso_status_inspector_options.png "VSO Options")

So if you like it download/contribute.

[**Download**](https://visualstudiogallery.msdn.microsoft.com/e87c82b9-dced-4fe2-9a40-f90139c56882)

[**Source Code**](https://github.com/onlyutkarsh/VSOStatusInspector/)

That's it. Simple and small extension. Hope it helps you to keep and eye on VSO status and helps you to quick glance of its status when it down.

 <a href="http://ctt.ec/5d5d7"><img src="http://clicktotweet.com/img/tweet-graphic-4.png" alt="Tweet: Inspect #VisualStudioOnline status inside #VisualStudio - http://ctt.ec/5d5d7+ developed by @onlyutkarsh @arora_tarun #vs2013 #vs2015"></a>