---
layout: post          #important: don't change this
title: "Subscribe to Windows Explorer close event"
date: 2013-06-29 16:49:38
author: utkarsh
tags: [windows, csharp, dotnet, shell]
categories:
- "Windows"
- "csharp"
- "dotnet"
- "Shell"
description: "Subscribe to Windows Explorer close event"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
Lately I am busy working on developing a small shell extension. There is a functionality in my shell extension, that user opens the .NET windows form by right clicking on a file (context menu). The user then decides to close the parent explorer window (instead of closing the windows form opened). Now, in such a case I would like to close the windows form too. So, to achieve this I had to subscribe to the close event of the windows explorer. The below code shows the how to achieve the same. Although I am monitoring window close (destroy event), this can done for any process in Windows for which you have the handler (For ex: Notepad). Read more at link provided in “References” section.

So we are going to do it by performing following steps:

*   Find out the handle of the window we are interested in 
*   Hook the Destroy event 
*   When the event is raised, perform the desired operation   

### Find out the handle of the window we are interested in ###

```cs
public static IntPtr Find(string moduleName, string mainWindowTitle)
{
    //Search the window using Module and Title
    IntPtr WndToFind = NativeMethods.FindWindow(moduleName, mainWindowTitle);
    if (WndToFind.Equals(IntPtr.Zero))
    {
        if (!string.IsNullOrEmpty(mainWindowTitle))
        {
            //Search window using Title only.
            WndToFind = NativeMethods.FindWindowByCaption(WndToFind, mainWindowTitle);
            if (WndToFind.Equals(IntPtr.Zero))
                return new IntPtr(0);
        }
    }
    return WndToFind;
}
```
We call the Find method as below. P.S: In my shell extension this window handler is automatically given by the library to me, so I do not use Find method.

```cs
string ModuleName = "explorer.exe";
    string MainWindowTitle = "Libraries";

    var targetWnd = Find(ModuleName, MainWindowTitle);
```

### Hook the Destroy event ###

Put the below code in your constructor or initialization method.

```cs
public MainWindow()
{
    InitializeComponent();
    Dictionary<AccessibleEvents, NativeMethods.WinEventProc> events = InitializeWinEventToHandlerMap();

    //Hook window close event - close our HoverContorl on Target window close.
    NativeMethods.WinEventProc eventHandler = new NativeMethods.WinEventProc(events[AccessibleEvents.Destroy].Invoke);
    GCHandle gch = GCHandle.Alloc(eventHandler);

    _garbageHook = NativeMethods.SetWinEventHook(AccessibleEvents.Destroy, AccessibleEvents.Destroy, IntPtr.Zero, eventHandler
    , 0, 0, NativeMethods.SetWinEventHookParameter.WINEVENT_OUTOFCONTEXT);
}

private Dictionary<AccessibleEvents, NativeMethods.WinEventProc> InitializeWinEventToHandlerMap()
{
    Dictionary<AccessibleEvents, NativeMethods.WinEventProc> dictionary = new Dictionary<AccessibleEvents, NativeMethods.WinEventProc>();
    //You can add more events like ValueChanged - for more info please read - 
    //http://msdn.microsoft.com/en-us/library/system.windows.forms.accessibleevents.aspx
    dictionary.Add(AccessibleEvents.Destroy, new NativeMethods.WinEventProc(this.DestroyCallback));

    return dictionary;
}
```

### When the event is raised, perform the desired operation ###

```cs
private void DestroyCallback(IntPtr winEventHookHandle, AccessibleEvents accEvent, IntPtr windowHandle, 
    int objectId, int childId, uint eventThreadId, uint eventTimeInMilliseconds)
{
    //Make sure AccessibleEvents equals to LocationChange and the current window is the Target Window.
    if (accEvent == AccessibleEvents.Destroy && windowHandle.ToInt32() == _targetWindowHandler.ToInt32())
    {
        //Queues a method for execution. The method executes when a thread pool thread becomes available.
        ThreadPool.QueueUserWorkItem(new WaitCallback(this.DestroyHelper));
    }
}

private void DestroyHelper(object state)
{
   MessageBox.Show("Good bye!");
    //Removes an event hook function created by a previous call to 
    NativeMethods.UnhookWinEvent(_garbageHook);

}
```

This demo application can be downloaded from below provided link, where in I am opening the explorer window and listening to close event. On close I am showing a simple message box.

[Download Code](https://github.com/onlyutkarsh/ExplorerCloseEventListener)

### References ###

[Add your control on top of another application – By Shai Raiten](http://www.codeproject.com/Articles/80255/Add-Your-Control-On-Top-Another-Application)