---
layout: post          #important: don't change this
title: "Better way to load dependent assemblies in your Visual Studio extensions"
date: 2014-12-13 16:46:56
author: utkarsh
tags: [Extensions, VisualStudio]
categories:
- "VisualStudio"
- "Extensions"
description: "Better way to load dependent assemblies in your Visual Studio extensions"
image:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
I had [previously](http://geekswithblogs.net/onlyutkarsh/archive/2013/06/02/loading-custom-assemblies-in-visual-studio-extensions-again.aspx) written on how to load custom assemblies in your extension using `AppDomain.CurrentDomain.AssemblyResolve`. It required few lines of code to be written in your VS package class. Today I am going to show you an easier way of doing the same. 

Visual Studio provides [ProvideBindingPath](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.providebindingpathattribute.aspx)`attribute which lets Visual Studio know other paths from where your extension loads the assemblies. The usage of this attribute is very simple, you just need to decorate your package class with it. 

```cs
[Guid(GuidList.guidmin2015PkgString)]
[ProvideBindingPath]
public sealed class min2015Package : Package
{
}
```

Once you do that and compile your extension, the <extension>.pkgdef file will be modified to include a new line something like below, where {PackageGuid} will be the guid of your package.

```cs
[$RootKey$\BindingPaths\{PackageGuid}]
```

So, when you install the extension, along with other information, this information is also written in the registry. So when Visual Studio is loading your extension, it will also load all the assemblies from the path you have mentioned in to its app domain.

### SubPath property

You can also decide to keep all your dependent assemblies in a separate folder under your extension folder. If you decide to do so, you need to mention that using `SubPath` property. Example below lets Visual Studio know that, the dependent assemblies are References subfolder.

```cs
[ProvideBindingPath(SubPath="References")]
```

That's it for now. Happy extending Visual Studio!