---
layout: post          #important: don't change this
title: "Get Assembly Full Name using context menu"
date: 2013-07-27 17:51:14
author: Utkarsh Shigihalli
categories:
- "csharp"
- "dotnet"
- "Shell"
- "Windows"
description: "Get Assembly Full Name using context menu"
img:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
### Assembly Full Name - What is it? ###
.NET assemblies contain full name, also known as fully qualified name. The full name is stored in the meta data of the assembly and it used by .net runtime to uniquely identify it from another with the same name. The fully qualified name is majorly used during assembly resolution and during registering http handlers. For more information on Assembly Full Name read [here](http://msdn.microsoft.com/en-us/library/k8xx4k69.aspx)

I have developed a small shell extension which adds a new context menu “Assembly Full Name” to .net assemblies. With this, you can view just the full name and also copy the same to clipboard. Below is the small demo video of its working as it does not need much of explanation. If this sounds useful to you, you can **download it below**.

### Demo ###

{% include youtubeplayer.html id="WM0MxaUTwTE" %}

### Download ###
<iframe src="https://onedrive.live.com/embed?cid=63D3FBCA39592C79&resid=63D3FBCA39592C79%218742&authkey=AOYwHPrbj-Y6MUs" width="98" height="120" frameborder="0" scrolling="no"></iframe>