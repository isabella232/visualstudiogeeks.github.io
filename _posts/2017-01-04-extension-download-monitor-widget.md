---
layout: post
title: "Extension Download Monitor Widget - Monitor your extension downloads/ratings from VSTS/TFS dashboard"
date: 2017-01-04
author: utkarsh
tags: ["Extensions"]
categories:
- "Extensions"
img: "/images/screenshots/utkarsh/extensiondownloadmonitor/widgets.png"
description: "Extension Download Monitor Widget"
permalink: extensions/extension-download-monitor-widget
keywords: "Extensions"
---


This dashboard widget allows you to track your extension's downloads, ratings and downloads/day right from the VSTS/TFS dashboard!

<!--more-->

## Features ##

- Track downloads/ratings of any extension - Visual Studio, VSTS/TFS extensions, and also Visual Studio Code extensions.
- Know how many users downloaded your extension today.
- Track the average rating and along with total number of ratings your extension has received.

![dashboard](/images/screenshots/utkarsh/extensiondownloadmonitor/widgets.png)

## Add to Dashboard ##

1. Follow the steps explained [here](https://www.visualstudio.com/en-us/docs/report/dashboards#add-a-widget) and select `Extension Download Monitor` widget.

## Configuration ##

1. Browse to extension page in Visual Studio Marketplace.
2. Look for `itemName=value` in the url as highlighted in screenshot below.

    ![itemName](/images/screenshots/utkarsh/extensiondownloadmonitor/itemname.png)

3. Get the `itemname` for your extension. For example item name is  `onlyutkarsh.ExportImportBuildDefinition` in the above screenshot.
4. Paste that in the `Item Name` text box in the configuration.

## Other Options ##

1. The extension supports 2 sizes (1x1 and 2x1).
2. You can also select color for the widget from various available colors.

![Configuration](/images/screenshots/utkarsh/extensiondownloadmonitor/configuration.png)

## Report Issues ##
Found an issue or want to suggest a feature? Add it as issue [here](https://github.com/onlyutkarsh/extensiondownloadmonitorwidget/issues).
