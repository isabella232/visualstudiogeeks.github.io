---
layout: post          #important: don't change this
title: "MSBuild Dependency Visualizer tool"
date: 2016-01-11 21:16:00 
author: utkarsh
tags: ["VisualStudio", "MsBuild", "Dependency Visualizer"]
categories:
- blog                #important: leave this here
- msbuild
- "visual studio"

img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---
My current client is still on TFS 2010 and is getting ready to upgrade to TFS 2015. One of the important task during the upgrade is to analyze your build dependencies. We have a lot of MSBuild (we have .proj extension) files which are triggered via our build definitions which have dependencies on other files. It becomes really hard to debug or understand the dependencies manually. I hence decided to write a small utility to visualize these dependencies.
<!--more-->
Take a look at the following screenshot of the tool.

![Alt text](/images/screenshots/utkarsh/msbuild_dependency_visualizer.png)

As you can see, the tool visualizes the dependencies in a nice connected graph. On selecting any node (color yellow), it highlights the parent (in light blue) and child (in pink color) dependencies. Also, clicking a node marks the parent links in blue and child links in red.

 
Please note, the tool is in early beta version and there are few **limitations/known issues**.

- The tool does not resolve properties.
- Scans only `Imports` in file, does not scan the targets.
- Exception handling is minimal.


**Future plans**

- Save the graph as image
- Ability to double click and open the file
- On demand scan of the targets within the file

For anyone curious, I am **using following components**

- WPF and [MahApps](http://mahapps.com/) for UI
- [Microsoft Automatic Graph Layout (MSAGL)](http://research.microsoft.com/en-us/projects/msagl/) library for drawing graph

You can download the source code from [GitHub](https://github.com/onlyutkarsh/MSBuildDependencyVisualizer/)