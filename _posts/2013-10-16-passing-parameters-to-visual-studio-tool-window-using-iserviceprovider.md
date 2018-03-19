---
layout: post          #important: don't change this
title: "Passing parameters to Visual Studio tool window using IServiceProvider"
date: 2013-10-16 20:00:20
author: utkarsh
tags: [Extensions, VisualStudio]
categories:
- "VisualStudio"
- "dotnet"
- "csharp"
- "Extensions"
description: "Passing parameters to Visual Studio tool window using IServiceProvider"
image:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
### Assumption

You have a toolwindow in your extension and show it on click of a menu/button/link as below.

```cs
ToolWindowPane windowPane = Package.FindToolWindow(typeof(TeamRoomToolWindow), 0, true);
var control = windowPane.Content as TeamRoomToolWindowContent;
if (control != null)
{
    var frame = windowPane.Frame as IVsWindowFrame;
    if (frame != null)
    {
        frame.Show();
    }
}
```
**OR**

```cs
IVsUIShell service = _serviceProvider.GetService(typeof(IVsUIShell)) as IVsUIShell;

if (service != null)
{
    IVsWindowFrame winFrame;
    var guidNo = new Guid(GuidList.MY_TOOL_WINDOW_GUID);
    if (service.FindToolWindowEx(0x80000, ref guidNo, 0, out winFrame) >= 0 && winFrame != null)
    {
        winFrame.Show();
    }
}
```

### Problem

You are following a MVVM model in your extensibility project and your Views (user controls, tool windows) used in the extension are most probably in different project than in the package. In such a case if is difficult to pass the parameter to toolwindow with the above code as it is a COM call.

### Solution

In this blog post I am going to show you how to create Visual Studio services so that the other parts of your solution can access the information.

**Introduction to services**: A service in Visual Studio is like a contract which you expose so that the parts of your extension is accessible to outside of the extension. Say for other extensions. You will be surprised to know that a large part of Visual Studio is made of services and majority of the communication happen through services. For example, output window, team explorer window are all exposed as services so that your package can interact with them. 

The documentation on Visual Studio services say that the Visual Studio packages which expose services are called as service provider packages. The definition and advantages of the services within package is a big topic in itself and lets cover that in another article. You can also refer to the documentation/example provided [here](http://archive.msdn.microsoft.com/ServicesRefDD/Release/ProjectReleases.aspx?ReleaseId=1215)

The point in this article is to just pass the information to other parts of the extension (which is different project than package) through Visual Studio services.

So lets see with an **example**:

My solution structure is as below

![image]({{site.url}}/images/screenshots/utkarsh/2013_10_16_passing_parameters_to_visual_Image1.png)

1.  Visual Studio package project 
2.  Team Explorer Integration project (not necessary for creating the service) 
3.  Core logic of the solution 

My current UI of the extension is such that I display a list of users in Team Explorer. The list is bound to the listview by a ObservableCollection. 

![image]({{site.url}}/images/screenshots/utkarsh//2013_10_16_passing_parameters_to_visual_Image2.png)

I have a requirement that I need to open a toolwindow on click of the user and display the name on a button. I have a command bound to the hyperlink in my ViewModel. As you can see in the above code, either way opening the toolwindow is a COM call and it is not possible to pass the parameter to toolwindow. The main reason is that `FindToolWindow` call is defined in Package and which is not accessible to us outside the pacakge project. For this purpose we will define the new service which will help us to open the toolwindow and pass the string clicked on the team explorer.

To give you an idea, below is the output which we are targeting.

![ToolBarDemo]({{site.url}}/images/screenshots/utkarsh/2013_10_16_passing_parameters_to_visual_Image3.gif)

1.  Create the interfaces for our service. We are going to need the 2 interfaces (`IToolWindowManager` and `SToolWindowManager`) to follow the VSSDK pattern of creating service. The latter will just act as a address to the `IToolWindowManager` service. 
2.  Implement the services in a separate class or the package (for this example I am going to implement in package itself) 
3.  Register the services in the during package initialization. 
4.  Expose the services to Visual Studio so that it loads the correct package to call the service. 

#### 1. Create the interfaces for our service.

```cs
[Guid("0C7C91F7-CA87-49D2-AE9E-BC1AEA81CC3C")]
[ComVisible(true)]
public interface IToolWindowManager
{
    void PassNameAndOpenToolWindow(string name);
}

[Guid("7061ACA0-AA8E-4A7B-8B8F-306CEB948200")]
public interface SToolWindowManager
{
}
```

#### 2. Implement the services in a separate class or the package

As you can see with the below code, my package class implements both the interfaces and implements the method defined. If you notice closely, I am accessing the control hosted in the toolwindow and setting the property defined in the control. Once you set the property its regular WPF on the job, which is binding and `INotifyPropertyChanged`.

```cs
public sealed class ToolWindowDemoPackage : Package, IToolWindowManager, SToolWindowManager
{
    public ToolWindowDemoPackage()
    {
        IServiceContainer serviceContainer = this;
        ServiceCreatorCallback creationCallback = CreateService;
        serviceContainer.AddService(typeof(SToolWindowManager), creationCallback, true);
    }
    private object CreateService(IServiceContainer container, Type serviceType)
    {
        if (container != this)
        {
            return null;
        }
        if (typeof(SToolWindowManager) == serviceType)
        {
            return this;
        }
        return null;
    }
    public void PassNameAndOpenToolWindow(string name)
    {
        ToolWindowPane windowPane = FindToolWindow(typeof(MyToolWindow), 0, true);
        var control = windowPane.Content as MyToolWindowContent;
        if (control != null)
        {
            var frame = windowPane.Frame as IVsWindowFrame;
            if (frame != null)
            {
                frame.Show();
            }
            control.ClickedName = name;
        }
    }
}
```

#### 3. Register the services in the during package initialization. 

```cs
public ToolWindowDemoPackage()
{
    IServiceContainer serviceContainer = this;
    ServiceCreatorCallback creationCallback = CreateService;
    serviceContainer.AddService(typeof(SToolWindowManager), creationCallback, true);
}

private object CreateService(IServiceContainer container, Type serviceType)
{
    if (container != this)
    {
        return null;
    }

    if (typeof(SToolWindowManager) == serviceType)
    {
        return this;
    }

    return null;
}
```

#### 4. Expose the services to Visual Studio so that it loads the correct package to call the service. 

This is simple and we just need to add following attribute to the package class which tell the Visual Studio Shell that this package exposes the service.

```cs
[ProvideService(typeof(SToolWindowManager))]
public sealed class ToolWindowDemoPackage : Package, IToolWindowManager, SToolWindowManager
{
}
```
That’s it. I know, it might seem too much to and also confusing at the first glance if code, but services are easy and powerful if you are a extension developer.

The demo code is pushed in the GitHub and you can download from [here](https://github.com/onlyutkarsh/ToolWindowDemo/)


> Note: The solution is built in VS2013 and you need VS2013 SDK installer to compile the project. Also, to reduce the size I have not uploaded the referenced SDK assemblies.**