---
layout: post          #important: don't change this
title: "Issues while extending Team Explorer in Visual Studio 2013"
date: 2013-09-14 18:23:37
author: Utkarsh Shigihalli
tags: [visualstudio, extensions]
categories:
- "Extensions"
- "dotnet"
- "VisualStudio"
description: "Issues while extending Team Explorer in Visual Studio 2013"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
If you have extended Team Explorer 2012 through your Visual Studio extension, I am sure you would have referred code from this [article](http://code.msdn.microsoft.com/vstudio/Extending-Explorer-in-9dccd594). The code in that article provides you all the necessary infrastructure to quickly integrate your extension with Team Explorer in Visual Studio 2012. However, with Visual Studio 2013 RC released, you will face an issue if you use the same base project to migrate your extension to Visual Studio 2013. So, in this blog post I will let you know what is the issue you will encounter, cause for it and how to resolve it.

### Issue ###

“InvalidCastException error in GetService<T> method”

**Additional information:** Unable to cast object of type `Microsoft.VisualStudio.Services.Integration.ContextManager` to type `Microsoft.TeamFoundation.Client.ITeamFoundationContextManager`

![image](/images/screenshots/utkarsh/2013_09_14_issues_while_extending_team_Image1.png)

### Cause ###

The project is referencing Visual Studio 2012 assemblies.

### Resolution ###

Refer to Visual Studio 2013 assemblies

1.  Remove the following assembly references from the TeamExplorerIntegration project.      

![image](/images/screenshots/utkarsh/2013_09_14_issues_while_extending_team_Image2.png)

2.  Refer to the Visual Studio 2013 assemblies. The above three assemblies are present in below path      

`C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\ReferenceAssemblies`

> [Update] For detailed steps on how to upgrade Visual Studio 2012 extension to Visual Studio 2013, please refer my friend Tarun's great [post](http://geekswithblogs.net/TarunArora/archive/2013/06/27/upgrading-vsix-extensions-from-vs2012-to-vs2013.aspx).  