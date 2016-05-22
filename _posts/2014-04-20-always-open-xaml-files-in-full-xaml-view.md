---
layout: post          #important: don't change this
title: "Always open XAML files in full XAML view"
date: 2014-04-20 19:23:08
author: utkarsh
tags: [VisualStudio, XAML, WPF]
categories:
- "XAML"
- "WPF"
description: "Always open XAML files in full XAML view"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
Lately in my current project, I have been doing lot of UI work and hence I constantly juggle between view model classes and Views (XAML files). When I double click the XAML files in Solution Explorer, it opens designer as well as XAML view.

![SNAGHTML232d54af](/images/screenshots/utkarsh//2014_04_20_always_open_xaml_files_Image1.png)

Of course you can double click XAML tab (circled in red in above screenshot) and expand xaml view. However, if you are like me and more comfortable working in full xaml editor, you can actually make editor open full XAML editor always.

Steps:

1.  Go to Tools –>Options to open Visual Studio Options window 
2.  Find XAML node and go to Miscellaneous node.      

![image](/images/screenshots/utkarsh//2014_04_20_always_open_xaml_files_Image2.png)

3.  Then check "Always open documents in full XAML view", then click OK   

Now, whenever you double click XAML files in Solution Explorer, your xaml files are opened in full XAML view.