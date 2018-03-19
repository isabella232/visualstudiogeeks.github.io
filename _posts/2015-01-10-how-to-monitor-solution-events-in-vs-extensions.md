---
layout: post          #important: don't change this
title: "HOWTO: Monitor solution events in Visual Studio extensions "
date: 2015-01-10 10:26:00 
author: utkarsh
tags: [Extensions, "VisualStudio"]
categories:
- blog                #important: leave this here
- "visual studio extensibility"
image:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---

Have you noticed, In Visual Studio, when we close the solution, all the build/other messages from the output window are cleared automatically?  How is it done? Well, in this post we will see just that. For example, we can monitor events such as `OnAfterLoadProject`, `OnAfterOpenSolution` etc and also take actions when they are triggered. 
<!--more-->
To show what we will achieve at the end of this blog, please look at the demo below.
 
![Solution Events]({{site.url}}/images/screenshots/utkarsh/2014-01-10-solutionevents.gif)

Intersting right? As you can see, we monitor for the events in the solution and log them in the custom toolwindow (called My Tool Window) in the extension. Lets start coding... 

Visual Studio SDK provides below two interfaces with which we can achieve this.

- [IVsSolution](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.ivssolution.aspx) 
- [IVsSolutionEvents](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.ivssolutionevents.aspx)

## Get IVsSolution instance ##

Through your package instance, you can call GetService method as below to get the instance of `IVsSolution`. 

```csharp
_solutionService = GetService(typeof(SVsSolution)) as IVsSolution;
if (_solutionService != null)
{
    Common.Instance.Package = this;
    Common.Instance.Solution = _solutionService;
}
```

In the above example, once I get the instance of IVsSolution, I am storing the package and solution instance, in a singleton class `Common`. This is utility class for this extension which will help us to access these instances from the view-model our toolwindow uses.

## Implement IVsSolutionEvents ##
As you can see in the demo above, I am showing all the events in a toolwindow. To do that, I am binding a collection to WPF listview with the event names. I have hence implemented this interface to the view-model itself. Ideally you would want to do this in a separate class.

```csharp
internal class MyControlViewModel : IVsSolutionEvents
{
	//rest of the code
}
```

## Register for notifications of solution events ##
In the constructor of the view-model, we would start listening for the solution events.

```csharp
public MyControlViewModel()
{
    uint cookie;
    EventsList = new ObservableCollection<string>();
    Common.Instance.Solution.AdviseSolutionEvents(this, out cookie);
    Common.Instance.SolutionCookie = cookie;
}
```

In the code above [AdviseSolutionEvents](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.ivssolution.advisesolutionevents.aspx) registers view-model class as listener. The `cookie` stores the pointer to the solution event object. We will store this instance `Common` class so that it can be accessed from Package class later. `cookie` is required for us to unadvise (unregister) the events later on.

## Implement IVsSolutionEvents methods ##

This interface provides [many methods](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.ivssolutionevents_methods.aspx) related to solution events. For keeping the blog post short, example method implementation below.

```csharp
public int OnAfterOpenProject(IVsHierarchy pHierarchy, int fAdded)
{
    EventsList.Add("Triggered OnAfterOpenProject");
    return VSConstants.S_OK;
}
```

Above code snippet, adds a message to the collection which is bound to the listview.

## Unregister from solution events notification ##
Finally, when package is unloaded or Visual Studio closed, we need to Unadvise from event notification. In the example, we will use the cookie stored in the `Common` class before and unadvise as below. I am doing this in Package class by overriding `Dispose` method.

```csharp
protected override void Dispose(bool disposing)
{
    base.Dispose(disposing);

    if (_solutionService != null && Common.Instance.SolutionCookie != 0)
    {
        _solutionService.UnadviseSolutionEvents(Common.Instance.SolutionCookie);
    }
}
```

That's it for this post. You can browse/download the code used to build this demo from [**GitHub**](https://github.com/onlyutkarsh/SolutionEventsMonitor/).