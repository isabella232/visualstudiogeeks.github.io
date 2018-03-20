---
layout: post
title: "Open sourcing Export/Import Build Definition extension"
date: 2017-02-23
author: utkarsh
tags: ["Extensions"]
categories:
- "Extensions"
image: "/images/screenshots/utkarsh/export-import-build-definition/context-menu-new2.png"
description: "Open sourcing Export/Import Build Definition extension"
permalink:
keywords: "Extensions"
---
I have decided to [open source](https://github.com/onlyutkarsh/ExportImportBuildDefinition) my favorite [**Export/Import build definition extension**](http://bit.ly/exportimportbuild). I wanted to open source the extension from day one, but at that time, I was just starting to learn the extension development and wrote the code hastily concentrating only on achieving the minimum functionality without much thought to code. So in the end my code was too complicated to understand, tightly coupled and hence difficult to manage.

<!--more-->

However, after publishing couple of [my extensions](https://marketplace.visualstudio.com/search?term=publisher%3A%22Utkarsh%20Shigihalli%22&target=VSTS&sortBy=Relevance), I now find myself bit comfortable with extensions development. I started writing my extensions using Visual Studio. However, I soon realized that I do not need a visual studio solution (.sln), project etc for VSTS extensions. I also found debugging of VSTS extensions in Visual Studio, a bit of struggle.

So I decided to switch to VSCode. Reasons for this switch were many, its free, cross-platform, great set of extensions, super light-weight etc. With VSCode, Webpack and its Debugger for Chrome extension, I think I have found my perfect dev environment for VSTS/TFS extension development. It allows me to debug in VSCode, [typescript linting](https://marketplace.visualstudio.com/items?itemName=eg2.tslint), have the local server using [Webpack Dev Server](https://webpack.github.io/docs/webpack-dev-server.html), [HMR](https://webpack.github.io/docs/webpack-dev-server.html#hot-module-replacement) and also allows me to package my code with [optimization](https://webpack.github.io/docs/optimization.html).  The credit for this source code structure should completely go to _Chris Mason_ of Microsoft. He was very helpful to me resolve all the issues and also shared examples while I was setting up my code. While I was busy experimenting the structure, there were few typescript updates. I decided to use the latest version of Typescript (the extension uses v2.1.6 supported by VSCode).

For all this to work, I had to completely rewrite my extension and in that process, refactored lot of code to clean and use the latest features of typescript. More importantly, I also made additional validations during import like, check for service endpoints, custom tasks, refined UI to show validation errors etc.

![Validations]({{site.url}}/images/screenshots/utkarsh/export-import-build-definition/Validations.png)

So I encourage anyone who is interested to bring enhancement to the extension or have ideas for making extension work better to take a look at the code and contribute. For those interested in contributing to this extension I would request to start with the [readme](https://github.com/onlyutkarsh/ExportImportBuildDefinition/blob/master/README.md).