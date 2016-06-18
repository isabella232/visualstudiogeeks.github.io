---
layout: post
title: "Export your GeeksWithBlogs.net blog posts as Markdown files"
date: 2016-06-18 
author: utkarsh 
tags: ["Tools", "WPF"]
categories:
- WPF
- Tools
img: "/images/screenshots/utkarsh/gwb2md/maniui.jpg"
description: "Export your GeeksWithBlogs.net blog posts as Markdown files"
keywords: "Geekswithblogs, Markdown, AzureDevTestLabs"
---

Do you host your blog posts in [GeeksWithBlogs.net](http://www.geekswithblogs.net) (GWB)? Well you probably know then that, GWB does not provide any means to download/backup your posts. This is big problem if if you want to move to another hosting provider like GitHub/Wordpress and you cannot download any your existing posts. Fortunately, GWB posts are in MetaWebBlog format and with some simple C# code you can backup your blog posts. 

I host my blog here using Jekyll engine and wanted a quick way to convert all my posts in the markdown format. This utility provides a quick way to fetch your posts from GWB and save them as markdown.

<!--more--> 

## Features ##

- View all your blog posts.
- Save individual posts or all as markdown files.
- Ability to add optional frontmatter (necessary if you are hosting in GitHub for example).
- Dynamically insert GWB author name, post date and categories.
- Download images in the posts
- GWB returns image name in the posts as `image001.jpg` - this tool automatically renames the images with the post tile slug (ex: postdate_in_yymmdd_post_title.jpg)
- Optionally overwrite image path in the markdown - might be useful when you host the images in root of your folder in your git repo and want to refer images with the full path.

## Walkthrough ##

1. Run the GeeksWithBlogsToMarkdown.exe
2. Enter your GWB blog details in settings
3. Click Get posts icon (first icon) from the Main window from top-right options
4. Click Save/Save all button to save the post as markdown

Once you click **save/save all** button, following operations are performed

- Download the images and rename them with the post slug.
- Markdown is generated
- Add any frontmatter after replacing any custom tags (`<$GWB_...$>`) with the actual values
- Replace the image URL with the downloaded images - If custom image path is specified that path is used.
  
## Screenshots ##

There are only two screens actually in the UI. As you can see it is very basic.

### Main UI ###

![Main UI](/images/screenshots/utkarsh/gwb2md/mainui.jpg)

1. Blog post tile and URL from GWB
2. Main menu options
	- Get Posts
	- Save this post as markdown
	- Save all posts as markdown
	- Settings
3. All the posts from GWB
4. The post HTML received from the GWB (readonly)
5. The equivalent markdown generated  (readonly)

### Settings ###

![Settings screen](/images/screenshots/utkarsh/gwb2md/settings.jpg)

1. Full URL of your GWB blog post
2. User name you use to login to admin console
3. Password for your admin account
4. Output folder where the saved markdown needs to be saved
5. Output folder for the downloaded images
6. Optional images path to write in the markdown files

### Limitations ###
1. Does not identify languages used in the code snippets. You need to manually add triple backticks and add any languages necessary.
2. If you have referred any of your GWB blog post, the URL will be retained as is. You need to replace them manually.

### Download ###

If you just would like to use the tool, download the latest version from [here](https://github.com/onlyutkarsh/GeeksWithBlogsToMarkdown/releases)

The code is also [open source](https://github.com/onlyutkarsh/GeeksWithBlogsToMarkdown) on GitHub and you can contribute/check the code on how it is done. Excuse me for code as it is not in a good state, but this was a quick tool developed for sole purpose of moving my blog posts.