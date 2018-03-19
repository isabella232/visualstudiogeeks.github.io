---
layout: post          #important: don't change this
title: "Administer Visual Studio Tags using TagAdmin Visual Studio extension"
date: 2014-12-22 00:25:00
author: tarun
tags: [Extensions, VisualStudio]
categories:
- blog                #important: leave this here
- TagAdmin
image:   #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---
 
Manage Work Item tags from right with in Visual Studio using the TagAdmin extension. Watch this 2 minute video to get a headstart...
<!--more-->
Tagging as a feature has been a hit in the Team Foundation Server / Visual Studio Online family of tools.

Work Item Tags allow you to associate identifiers to work items to group search and filter work items easily. Tags can be added directly from the work item form or using the excel plug in. Adding tags is so easy that they start to grow in no time. While that's a good thing you may not want people to create multiple tags that refer to the same thing. For example, BackOffice and Administration may have been created by users to refer to the same idea. However, these are now 2 separate entities, you will not be able to bring back work items tagged to BackOffice if searching exclusively for work items tagged to Administration. It’s also very easy to end up with tags that are misspelled.

So, the problem as it stands out today is... How do you manage tags in projects and project collections? 

We are very excited to announce the release of our latest Visual Studio Extension "TagAdmin"... The extension uses the all new REST APIs for retrieving and updating the data in TFS/VSO. The extension makes use of Integrated authentication to ensure all communication between the client & server is through an authorized token flow. The latest version of the extension can be downloaded from the Visual Studio Extension Gallery [here](https://visualstudiogallery.msdn.microsoft.com/057740ff-86be-4a44-b2bb-7e6d68b75455). 

The first version of the extension allows you to,

    1.	View Active tags 
	2.	Rename tag
	3.	Delete tags 
	4.	View work items linked to tags
	5.	Supports both VSO and TFS 
	6.	Uses Integrated Authentication


Watch this 2 minute video of what the extension allows you to do today... 
[![Video](http://i.imgur.com/yqtKgjk.png)](https://www.youtube.com/watch?v=J4g_VJlVaxY&feature=youtu.be)

If you like our work, we would love for you to spread the word for us by tweeting via button below... 

<a href="http://ctt.ec/dZDaj"><img src="http://clicktotweet.com/img/tweet-graphic-4.png" alt="Tweet: Manage your workitem tags inside #VisualStudio using #TagAdmin #extension http://ctt.ec/dZDaj+ developed by @onlyutkarsh @arora_tarun" /></a>

We are still hard at work, do you have any ideas on how we can improve the extension? We would love to hear from you...Mail us at `visualstudiogeeks (at) gmail.com`
