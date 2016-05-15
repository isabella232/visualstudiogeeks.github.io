---
layout: post          #important: don't change this
title: "How to open web page in internal browser of Visual Studio"
date: 2013-12-06 16:57:34
author: Utkarsh Shigihalli
tags: [Extensions, VisualStudio]
categories:
- "extensions"
- "Shell"
- "VisualStudio"

description: "How to open web page in internal browser of Visual Studio"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
Tarun and I have started working on new Visual Studio extension, which we plan to release in couple of weeks after Christmas. One of the first thing I do as soon as we start the work on new tool/extension, is to gather all the features we have planned and build as many sample extensions as I can (each extension doing part of the functionality which is demo-able). This helps me to get the sense of complexity and also minimal code needed for a functionality to work. Another benefit is, we get the advantages and limitations of the implementation. This helps us to look for other alternatives if the PoC doesn't fit our need or too complex to build in the given time frame.

As part of this new extension, we need to open a web page on click of a hyperlink. But we do not like it to be opened in the external browser, rather we wanted it to be opened within Visual Studio itself. What I thought as a complex functionality, turned out to be too simple. This is pretty easy to implement with only 3 lines of code.

Visual Studio SDK has [IVsWebBrowsingService](http://msdn.microsoft.com/en-us/library/Microsoft.VisualStudio.Shell.Interop.IVsWebBrowsingService.aspx) interface, which has Navigate method. This will open the URL within Visual Studio itself.

```cs
IVsWindowFrame ppFrame;
var service = Package.GetGlobalService(typeof (IVsWebBrowsingService)) as IVsWebBrowsingService;
service.Navigate("http://www.visualstudio.com/", 0, out ppFrame);
```

The second parameter in the Navigate method takes [__VSWBNAVIGATEFLAGS](http://http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.__vswbnavigateflags.aspx) enum. I am passing zero, which force creates browsing tab if doesn't exist already and uses if it exists. If you want to create new tab every time the link is pressed you can use `__VSWBNAVIGATEFLAGS.VSNWB_ForceNew` in 2nd parameter of Navigate method.

<div id="scid:5737277B-5D6D-4f48-ABFC-DD9C333F4C5D:39be9de3-1c91-4736-a851-374bbbc12308" class="wlWriterEditableSmartContent" style="width: 448px; float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto"><div><object width="500" height="300"><param name="movie" value="http://www.youtube.com/v/TbAsLvWWkmM?hl=en&hd=1"><embed src="http://www.youtube.com/v/TbAsLvWWkmM?hl=en&hd=1" type="application/x-shockwave-flash" width="500" height="300"></object></div><div style="width:500px;clear:both;font-size:.8em">Demo</div></div>

Also, I have put a sample project file for you to quickly run and see how it works. You can download it from here - [https://github.com/onlyutkarsh/LaunchUrlDemo/](https://github.com/onlyutkarsh/LaunchUrlDemo/)