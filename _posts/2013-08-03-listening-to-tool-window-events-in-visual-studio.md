---
layout: post          #important: don't change this
title: "Listening to tool window events in Visual Studio"
date: 2013-08-03 12:58:10
author: Utkarsh Shigihalli
tags: [Extensions, VisualStudio]
categories:
- "VisualStudio"
- "csharp"
- "dotnet"
- "extensions"
description: "Listening to tool window events in Visual Studio"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
In this blog post we will see how to listen to toolwindow events. I am going to assume that you already know how to create toolwindows.

Tool windows are most common and widely used while you to work inside Visual Studio. If you are wondering what are tool windows inside Visual Studio, think about Solution Explorer, Immediate Window, Output Window and even Property Window. These windows are most flexible allowing you to dock, auto-hide and even support multiple monitors. 

![toolwindow](/images/screenshots/utkarsh//2013_08_03_listening_to_tool_window_Image1.jpg) 

If you are Visual Studio extension developer, sooner or later you will be required to create tool windows in your extensions, because they integrate seamlessly inside Visual Studio. 

[Visual Studio SDK](http://www.microsoft.com/en-in/download/details.aspx?id=30668) provides [IVsWindowFrameNotify3](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.ivswindowframenotify3%28v=vs.110%29.aspx) interface which needs to be implemented by your ToolWindowPane class. The interface IVsWindowFrameNotify3 provides methods to track close, change in dock state etc. 

> **Note**: Visual Studio fires `OnClose` event but does not actually close it. Instead it just hides it from the view. The actual Close event is a overridden method which is fired only when Visual Studio instance is closed.

<br/>

```cs
[Guid("6912BAC8-BD13-4B64-A675-D83902A63077")]
public class MyToolWindow : ToolWindowPane, IVsWindowFrameNotify3
{
    /// <summary>
    /// Standard constructor for the tool window.
    /// </summary>
    public MyToolWindow(): base(null)
    {
    }

    public int OnClose(ref uint pgrfSaveOptions)
    {
        return Microsoft.VisualStudio.VSConstants.S_OK;
    }

    public int OnDockableChange(int fDockable, int x, int y, int w, int h)
    {
        return Microsoft.VisualStudio.VSConstants.S_OK;
    }

    public int OnMove(int x, int y, int w, int h)
    {
        return Microsoft.VisualStudio.VSConstants.S_OK;
    }

    public int OnShow(int fShow)
    {
        return Microsoft.VisualStudio.VSConstants.S_OK;
    }

    public int OnSize(int x, int y, int w, int h)
    {
        return Microsoft.VisualStudio.VSConstants.S_OK;
    }
}
```