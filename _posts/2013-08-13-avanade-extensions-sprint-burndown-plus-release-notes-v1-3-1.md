---
layout: post          #important: don't change this
title: "Avanade Extensions - Sprint Burndown Plus Release Notes v1.3.1"
date: 2013-08-13 16:35:30
author: Utkarsh Shigihalli
tags: [Extensions, VisualStudio, burndown]
categories:
- "Burndown+"
- "Sprint"
- "extensions"
- "VisualStudio"
description: "Avanade Extensions - Sprint Burndown Plus Release Notes v1.3.1"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
Download – [Avanade Extensions - Burndown+](http://visualstudiogallery.msdn.microsoft.com/591f2516-9aec-4892-be08-53c1d63bc5a1)

### Problem ###

The Sprint burn down for few users would plot incorrectly. As you can see in the sample screen shot below, the sprint burn down has plotted like a [sawtooth wave](http://en.wikipedia.org/wiki/Sawtooth_wave). Though the sprint burn down chart is expected to be plotted as shown in the screen shot on the right. 

|Actual|Expected|
|---|---|
|![image](/images/screenshots/utkarsh//2013_08_13_avanade_extensions_-_sprint_Image1.png "image")|![image](/images/screenshots/utkarsh//2013_08_13_avanade_extensions_-_sprint_Image2.png "image")|
  
### Root cause ###

We have found the invalid code logic in our extension that is causing the saw tooth like plotting of the sprints. It would only ever happen to teams where there is a gap between the sprint end date and sprint start date. Not that having a gap is in correct, it’s just that our logic wasn’t handling this correctly. We have fixed the bug and have rolled out the fix in v1.3.1. 

### Next Steps ###

We would encourage you to download the update and continue using Avanade Extensions, we have been overwhelmed by the positive feedback we received from the community. Please keep the feedback coming… We are listening!!!