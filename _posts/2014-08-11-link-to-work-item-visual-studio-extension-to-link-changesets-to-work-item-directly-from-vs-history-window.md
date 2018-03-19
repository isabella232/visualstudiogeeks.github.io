---
layout: post          #important: don't change this
title: "Link To Work Item – Visual Studio extension to link changeset(s) to work item directly from VS history window"
date: 2014-08-11 01:32:50
author: utkarsh
tags: [VisualStudio, Extensions, workitem]
categories:
- "VisualStudio"
- "Extensions"

description: "Link To Work Item – Visual Studio extension to link changeset(s) to work item directly from VS history window"
image:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
By linking work items and other objects, you can track related work, dependencies, and changes made over time. As the following illustration shows, specific link types are used to track specific work items and actions. (– via MSDN)

While making a check-in, Visual Studio 2013 provides you a quick way to search and assign a work item via pending changes section in Team Explorer. However, if you forget to assign the work item during your check-in, things really get cumbersome as Visual Studio does not provide an easy way of assigning. For example, you usually have to open the work item and then link the changeset which involves approx. 7-8 mouse clicks. Now, you will really feel the difficulty if you have to assign work item to multiple changesets, you have to repeat the same steps again. 

Hence, I decided to develop a small Visual Studio extension to perform this action of linking work item to changeset bit easier. 

## How to use the extension?

First, download and install the extension from [VS Gallery](http://visualstudiogallery.msdn.microsoft.com/af28fbc6-e90e-4f06-94d0-21c8bbac9685) (Supports VS 2013 Professional and above).

Once you install, you will see a new "*Link To Work Item*" menu item when you right click on a changeset in history window.

![image]({{site.url}}/images/screenshots/utkarsh//2014_08_11_link_to_work_item_Image1.png "image")

Clicking *Link To Work Item* menu, will open a new dialog with which you can search for a work item. As you can see in below screenshot, this dialog displays the search result and also the type of the work item.

![image]({{site.url}}/images/screenshots/utkarsh//2014_08_11_link_to_work_item_Image2.png "image")
You can also open work item from this dialog by right clicking on the work item and clicking 'Open'. Finally, clicking Save button, will actually link the work item to changeset.

![image]({{site.url}}/images/screenshots/utkarsh//2014_08_11_link_to_work_item_Image3.png "image")

One feature which I think helpful, is **you can select multiple changesets** from history window and assign the work item to all those changesets. 

To summarize the features

1.  Directly assign work items to changesets from history window 
2.  Assign work item to multiple changesets 
3.  Know the type of the work item before assigning. 
4.  Open the work item from search results 
5.  It also supports all default Visual Studio themes.   

Below is a small **demo** showcasing the working of this extension. 

![demo]({{site.url}}/images/screenshots/utkarsh//2014_08_11_link_to_work_item_Image4.gif "demo")

Finally, if you like the extension, do not forget to rate and review the extension in [VS Gallery](http://visualstudiogallery.msdn.microsoft.com/af28fbc6-e90e-4f06-94d0-21c8bbac9685). Also, do not hesitate to provide your suggestions, improvements and any issues you may encounter via [github](https://github.com/onlyutkarsh/LinkToWorkItem/issues).