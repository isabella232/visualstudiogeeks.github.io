---
layout: post          #important: don't change this
title: "How to open Configuration Manager in Visual Studio extensions"
date: 2014-12-24 10:26:00
author: utkarsh
tags: [Extensions, VisualStudio]
categories:
- blog                #important: leave this here
- "visual studio extensibility"
 
image:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---
 
Imagine you are developing a Visual Studio extension which requires you to prompt users to modify their project build configurations. The ideal way to do that is to open default configuration manager window of Visual Studio and let users configure. In this post we will see how to open Configuration Manager window of Visual Studio via Visual Studio SDK. 
<!--more-->
![Alt text]({{site.url}}/images/screenshots/utkarsh/vs_config_manager.png "Optional title")

Visual Studio provides [IVsConfigurationManagerDlg](http://msdn.microsoft.com/en-us/library/vstudio/microsoft.visualstudio.shell.interop.ivsconfigurationmanagerdlg.aspx) interface which has a single method called `ShowConfigurationManagerDlg`. The method has no parameters and it is as easy as calling a `MessageBox.Show()` in windows forms. 

So to use this method, you need to first get the service instance using `GetService(...)` method from `Package` class as below.

```csharp
var configManager = GetService(typeof (SVsConfigurationManagerDlg)) as IVsConfigurationManagerDlg;
```

Once you get the instance of `IVsConfigurationManagerDlg` interface, you will call `ShowConfigurationManagerDlg` method to show configuration manager dialog as below.

```csharp
if (configManager != null)
{
    configManager.ShowConfigurationManagerDlg();
}
```

### Showing Configuration Manager window only when project is loaded ###

With the above code, I can open configuration manager window even when no project is loaded in Visual Studio, which basically opens up the configuration manager window without any configuration. In most cases that is not intended. 

We can tackle this problem in two ways.

1. Make our package load only when Visual Studio has project loaded.
2. Check whether any projects are loaded before displaying the configuration manager dialog.

In this post we will use option 2.

For this, we need to get services of another interface [IVsSolution](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.ivssolution.aspx). With this interface, we can request to get project count property, which returns us the number of projects currently loaded in to the environment.

```csharp
var solutionService = GetService(typeof(SVsSolution)) as IVsSolution;
object projectCount = null;
if (solutionService != null)
{
    solutionService.GetProperty((int)__VSPROPID.VSPROPID_ProjectCount, out projectCount);
}
if (projectCount != null && (int)projectCount <= 0)
{
    MessageBox.Show("No projects are open!");
}
else
{
	//open the configuration manager
}
```

With this, we show the configuration manager only when the environment has at least one project.

The demo code is uploaded in [GitHub](https://github.com/onlyutkarsh/OpenConfigurationManager) for your reference.
