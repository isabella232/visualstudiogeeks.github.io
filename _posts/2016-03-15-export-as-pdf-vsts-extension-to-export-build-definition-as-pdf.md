---
layout: post          #important: don't change this
title: "Export as PDF - VSTS extension to export build definition as PDF"
date: 2016-03-15 10:00:00 
author: Utkarsh Shigihalli
categories:
- blog                #important: leave this here
- extensibility
- "visual studio team services"
- "vsts"
description: "Export as PDF is a VSTS extension to export your build definition as PDF."
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshots/thumbs/
---
Ever wanted to see all the details of your build definition including its steps, triggers, history etc in a neat report so that you can print it or share it with a colleague? Well, now you can. Just export the build definition as PDF and get everything printed in a neat report. No need to click individual build steps to see build step arguments, also no need to browse individual tabs to see triggers, history etc.
<!--more-->

![Report](/images/screenshots/utkarsh/build2pdf.png)

## Get Started

> **Get it from Marketplace**: [http://bit.ly/exportaspdf](http://bit.ly/exportaspdf)

**Note:** The extension only supports new (non XAML) build defintions.

Once you install the extension, go to `Builds` hub and right click on any build definition. You will see a new menu item `Export as PDF`. Click on it and you will be able to select what to include in the report. Seclect the sections you want to be included in the PDF and click `Export as PDF`. The extension will scans your build definition, gets the details and generates the PDF which can be saved to your drive.

![Menu](/images/screenshots/utkarsh/menu_options.png)

## What you can export?
You can export following properties of the build definition.

- Build steps
- Variables
- Triggers
- History

## Limitations/Known issues
Due to VSTS REST API still being in a `preview`, following build defintion properties cannot be exported as of now. They will be supported in future versions of the extension.

- Options
- Repository information
- General tab

## Report Issues
Found an issue or want to suggest a feature? Add them at [http://bit.ly/exportaspdfissues](http://bit.ly/exportaspdfissues)

## Share
If you like this extension, please share with your colleagues and spread a word with a [tweet](http://twitter.com/home/?status=http%3A%2F%2Fbit.ly%2Fexportaspdf%20-%20Check%20this%20out!%20This%20%23vsts%20%23extension%20by%20%40onlyutkarsh%20exports%20ur%20build%20definition%20into%20a%20nice%20printable%20PDF%20report)!
