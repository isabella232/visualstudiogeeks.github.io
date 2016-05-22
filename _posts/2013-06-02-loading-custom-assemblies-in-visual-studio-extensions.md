---
layout: post          #important: don't change this
title: "Loading custom assemblies in Visual Studio extensions"
date: 2013-06-02 10:26:00
author: utkarsh
tags: [Extensions, VisualStudio]
categories:
- "VisualStudio"
- "extensions"
description: "Loading custom assemblies in Visual Studio extensions"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
If you ever tried to use/load third party assemblies inside your Visual Studio add-in or extensions, there is a good chance that you would have ran in to exception. This is because, Visual Studio extensions are hosted by Visual Studio process and they run in Visual Studio’s app domain. Extension on their own do not have their application domains. However, there is a bright side and you can still load the assemblies you want. The trick is that, when Visual Studio tries to load the assemblies it tries to resolve the assemblies which installed extensions are dependent upon. Hence, during this time, the Visual Studio fires [AssemblyResolve](http://msdn.microsoft.com/en-us/library/system.appdomain.assemblyresolve.aspx) event. So below are the steps to dynamically resolve the third-party assembly path and help Visual Studio resolve them.

Inside the `YourExtensionPackage.cs` constructor, register for the `AssemblyResolve` event.   

```cs
AppDomain.CurrentDomain.AssemblyResolve += OnAssemblyResolve;
```

Inside the `OnAssemblyResolve` method, resolve the path of the current assembly path and load the assembly

```cs
private Assembly OnAssemblyResolve(object sender, ResolveEventArgs args)
{
     string path = Assembly.GetExecutingAssembly().Location;
     path = Path.GetDirectoryName(path);

     if (args.Name.ToLower().Contains("telerik.windows.controls.gridview"))
     {
            path = Path.Combine(path, "telerik.windows.controls.gridview.dll");
            Assembly ret = Assembly.LoadFrom(path);
            return ret;
     }
     if (args.Name.ToLower().Contains("telerik.windows.controls.input"))
     {
            path = Path.Combine(path, "Telerik.Windows.Controls.Input.dll");
            Assembly ret = Assembly.LoadFrom(path);
            return ret;
     }
     return null;
}
```
Few points to note here though:

- In the above code `path` resolves to the path where the main extension assembly exist. 
- It is possible that you can even merge these assemblies inside the main extension assembly. Please read more in this [post](http://blogs.msdn.com/b/microsoft_press/archive/2010/02/03/jeffrey-richter-excerpt-2-from-clr-via-c-third-edition.aspx). I am yet to try this method in my extensions. 
- This trick works in other applications types as well. For example: Shell extensions, where host domain is of Windows Explorer (explorer.exe) process. 