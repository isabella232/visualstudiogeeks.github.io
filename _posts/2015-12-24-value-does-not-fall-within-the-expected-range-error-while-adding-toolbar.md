---
layout: post          #important: don't change this
title: "'Value does not fall within the expected range' error while adding toolbar to toolwindow"
date: 2015-12-24 22:16:00 
author: utkarsh
tags: [VisualStudio, Extensions, Exception]
categories:
- blog                #important: leave this here
- "extensibility"
- "visual studio"
image:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---

Hello, this is going to be a small post. In one of my Visual Studio extension, I was trying to add a toolbar to the toolwindow. Everything compiled just fine and all seemed perfect. However, as soon as I tried to open the toolwindow (with toolbar) I was getting "*Value does not fall within the expected range*" error.
<!--more-->

 ![Alt text]({{site.url}}/images/screenshots/utkarsh/value_does_not_fall_error.png)

Obviously, error did not give me much details on what caused the error. *Break on CLR exceptions* setting lead me to this line. 
 
```csharp
ToolWindowPane window = this.package.FindToolWindow(typeof(MyToolWindow), 0, true);
```


So, Assuming an issue with my declarions of toolwindow, I verified GUID's used, other declarations made in my VSCT file. All looked correct but still the same error.

After numerous debug sessions, **found out that Visual Studio throws this error when the toolbar you are adding does not have any command added to it. *That is, whenever you add a toolbar to the toolwindow, make sure you add at least one button (command) to it.***

So adding a `Button` element resolved the issue for me.