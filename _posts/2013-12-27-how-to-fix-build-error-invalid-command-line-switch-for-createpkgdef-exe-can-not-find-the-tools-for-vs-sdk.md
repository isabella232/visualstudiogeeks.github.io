---
layout: post          #important: don't change this
title: "How to fix build error Invalid command line switch for CreatePkgDef.exe. Can not find the tools for VS SDK"
date: 2013-12-27 19:21:45
author: Utkarsh Shigihalli
tags: [Extensions, VisualStudio]
categories:
- "VisualStudio"
- "extensions"

description: "How to fix build error Invalid command line switch for CreatePkgDef.exe. Can not find the tools for VS SDK"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
> I found this solution in the [MSDN forum](http://social.msdn.microsoft.com/Forums/vstudio/en-US/74158536-87be-4796-95d5-2b0c00ca6fc8/error-building-visual-studio-2012-vspackage-using-hosted-build?forum=TFService). I am merely posting it in my blog because I made this mistake twice and also I thought I should just document it here :-)

## Problem: ##

You are building a Visual Studio package and using hosted build server to build your code. When you build you get following error.

![image](/images/screenshots/utkarsh//2013_12_27_how_to_fix_build_Image1.png "image")

## Cause: ##

I initially thought build server does not have VS SDK installed. However, thanks to the answer by Jeff in this [forum question](http://social.msdn.microsoft.com/Forums/vstudio/en-US/74158536-87be-4796-95d5-2b0c00ca6fc8/error-building-visual-studio-2012-vspackage-using-hosted-build?forum=TFService) I was able to know the actual cause. I am quoting the below text from the forum answer

> The TFS build service runs as a 64-bit process on a 64-bit machine, and the VS SDK always gets installed as 32-bit (to work with Visual Studio). Since the registry key that the build process needs is located under `HKLM\SOFTWARE\WoW6432Node\Microsoft\VisualStudio\VSIP\11.0` the 64-bit build process looks for it in the wrong location.

## Solution: ##

*   Open the build definition which is causing the issue and go to "Process" tab 
*   Under the "Advanced" node, you will see "MSBuild Platform" 
*   Change it from "Auto" to "X86"   

![image](/images/screenshots/utkarsh//2013_12_27_how_to_fix_build_Image2.png)