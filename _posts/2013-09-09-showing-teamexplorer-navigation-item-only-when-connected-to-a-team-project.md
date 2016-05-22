---
layout: post          #important: don't change this
title: "Showing TeamExplorer Navigation Item only when connected to a Team Project"
date: 2013-09-09 14:07:45
author: utkarsh
tags: [Extensions, VisualStudio]
categories:
- "VisualStudio"
- "dotnet"
- "extensions"
description: "Showing TeamExplorer Navigation Item only when connected to a Team Project"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
Team Explorer in Visual Studio 2012 displays many context sensitive navigation items and links. If you would like to know what are navigation items and links take a look at the below picture.

![image](/images/screenshots/utkarsh/2013_09_09_showing_teamexplorer_navigation_item_Image1.png)

The team explorer shows certain navigation items (for example: My Work, Pending Changes etc) only when you are connected to TFS. These items are hidden (for obvious reasons) when you are not connected. 

In this blog post I will show you how to to hide/show the your custom navigation items in team explorer similar to other default navigation items. For how to extend Team Explorer (to create your own navigation items, links), please go through [this](http://code.msdn.microsoft.com/vstudio/Extending-Explorer-in-9dccd594) post by Microsoft.

- In the [Invalidate](http://msdn.microsoft.com/en-IN/library/microsoft.teamfoundation.controls.iteamexplorernavigationitem.invalidate.aspx) method of your Navigation Item class get the instance of the [ITeamFoundationContextManager](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.client.iteamfoundationcontextmanager.aspx).      

```cs
var teamFoundationContextManager = _serviceProvider.GetService(typeof(ITeamFoundationContextManager)) as ITeamFoundationContextManager;
```

- Validate returned context to see if it has [HasTeam](http://msdn.microsoft.com/en-IN/library/microsoft.teamfoundation.client.iteamfoundationcontext.hasteam.aspx) = true. HasTeam property will be true only when connected to a team project. If it is false (meaning, we are not connected to TFS), we set the visibility of our navigation item to false. 

```cs
if (teamFoundationContextManager != null && teamFoundationContextManager.CurrentContext != null)
    {
        if (teamFoundationContextManager.CurrentContext.HasTeam)
        {
            IsVisible = true;
        }
        else
        {
            IsVisible = false;
        }
    }
```

That’s it! With just by above code, your navigation item will be shown only when you are connected to a team project.