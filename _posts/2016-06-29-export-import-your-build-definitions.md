---
layout: post
title: "Export and Import your build definitions in VSTS and TFS between team projects"
date: 2016-07-01
author: utkarsh 
tags: ["Extensions", "VSTS"]
categories:
categories:
- extensibility
- "visual studio team services"
- "vsts"
image: "/images/screenshots/utkarsh/export-import-build-definition/small-context-menu.png"
description: "Export your GeeksWithBlogs.net blog posts as Markdown files"
keywords: "Extensions, VSTS"
---

**This VSTS/TFS extension will help you to export your build definition and then import it in same or another team project.**

Have you created a complex build definition with numerous steps, configured it with various build schedules, variables, and other build options? You now have a situation to create similar build definition in another team project - What do you do?

As you probably know there is no built-in way to do that currently and you have to manually create complete build definition again. But with this extension now you can with few simple steps!

<!--more-->

![Context Menu]({{site.url}}/images/screenshots/utkarsh/export-import-build-definition/context-menu.png)

Download from Visual Studio Marketplace: [http://bit.ly/exportimportbuild](http://bit.ly/exportimportbuild)

## Get Started ##

> **Note:** The extension only supports new (non XAML) build definitions.

Once you install the extension, go to `Builds` hub and right-click on any build definition. You will see a two new menu items `Export` and `Import`. You can also click on `All build definitions` on VSTS to see these menu items. 

![All Definitions Menu]({{site.url}}/images/screenshots/utkarsh/export-import-build-definition/small-context-menu.png)

### Export build definition ###

- Right-click and click `Export` on the build definition you would like to export.
- You will be prompted save the definition as a file.
- Save the file with JSON extension.

### Import build definition ### 

- Right click on `All build definitions` or any existing build definitions and click on `Import`
- You will be prompted to upload a build definition file. 
	![Import Dialog]({{site.url}}/images/screenshots/utkarsh/export-import-build-definition/import-dialog.png)
- You can either drag and drop a file or use the `Browse` button to select the file you want to import.
- Click `Import`. If import is successful, you will have your build created, with all your steps, variables, schedules and other build definition parameters.

![DefinotionCopy]({{site.url}}/images/screenshots/utkarsh/export-import-build-definition/definition.png)

## Limitations/Known issues ##

- Known issues on **On-Prem TFS**
	- If you are on on-prem TFS, `Export` and `Import` menu items only appear on build definition context menu and do not appear on `All build definitions` context menu.
	- If you do not have any existing build definitions, you need to create a temporary empty build definition to see these menu items.
- You will get an error if you trying to import a build definition whose associated team project or repository **does not exist** in the collection/account you are trying to import.
> TFS.WebApi.Exception: TF200016: The following project does not exist: `ProjectName`. Verify that the name of the project is correct and that the project exists on the specified Team Foundation Server.
**PS:** I am investigating correct way to resolve this issue and will fix this in future versions.

## Report Issues ##

Found an issue or want to suggest a feature? Add them at [http://bit.ly/exportimportbuildissues](http://bit.ly/exportimportbuildissues)