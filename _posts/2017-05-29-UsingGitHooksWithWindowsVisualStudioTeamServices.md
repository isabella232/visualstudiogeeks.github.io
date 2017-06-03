---
layout: post
title: "Using GitHooks with Visual Studio Team Services on Windows"
date: 2017-05-29
author: tarun
tags: ["DevOps", "GitHooks"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/ClientSideGitHooksGitWindowsVSTS.PNG"
description: "See how you can leverage GitHooks on client side in Windows for a repository backed up VSTS to automate quality inspection in your commits among other use cases you could apply GitHooks to in your DevOps solution... "
permalink: /DevOps/UsingGitHooksWithVstsGitOnWindows
published: true
keywords: "DevOps, GitHooks, GitHooks on Windows, GitHooks with VSTS, SourceControl, VersionControl, GitAutomation, GitHooks PowerShell, Git client side Hooks, CD with Git"
---
Ryan Hellyer accidentally leaked his Amazon AWS access keys to GitHub and woke up to a $6,000 bill the next morning. Wouldn't you just expect the source control as clever as git to just stop you from making such a blender?! Well, in case you didn't know you could put Git Hooks to work to address not just this but many similar scenarios...     
<!--more--> 

> “Git and continuous delivery is one of those delicious "chocolate & peanut butter" combinations we occasionally find in the software world – two great tastes that taste great together…” 

Well continuous delivery demands a significant level of automation… You can’t be continuously delivering if you don’t have a quality codebase. This is where git fares so well, it gives you the ability to automate most of the checks in your code base even before committing the code into you local repository let alone the remote. 

### What are GitHooks 
Git hooks allow you to run custom scripts whenever certain important events occur in the Git life-cycle, such as committing, merging, and pushing. Git ships with a number of sample hook scripts in the `repo\.git\hooks` directory. More information on Git Hooks is available [here](https://git-scm.com/book/gr/v2/Customizing-Git-Git-Hooks) 

### Practical use cases for using GitHooks 
Since GitHooks simply execute the scripts on the specific event type they are called on, you can do pretty much anything with Git hooks. Some examples of where you can use hooks to enforce policies, ensure consistency, and control your environment... 
- In Enforcing preconditions for merging 
- Verifying work Item Id association in your commit message
- Preventing you & your team from committing faulty code 
- Sending notifications to your team’s chat room (Teams, Slack, HipChat, etc)

### So where do I start?
Let's start by exploring client side Git Hooks... Navigate to `repo\.git\hooks` directory, you'll find that there a bunch of samples, but they are disabled by default. For instance, if you open that folder you’ll find a file called `pre-commit.sample`. To enable it, just rename it to `pre-commit` by removing the `.sample` extension and make the script executable. When you attempt to commit using `git commit`, the script is found and executed. If your pre-commit script exits with a 0 (zero), you commit successfully, otherwise the commit fails. 

![Client side GitHooks](/images/screenshots/tarun/GitHooksOnWindowsVSTS.PNG)

### GitHooks. Oh Windows!
Now if you are on windows, simply renaming the file won't work. Git will fail to find shell in the designated path as specified in the script. The problem was lurking in the first line of the script, the __shebang__ declaration:

``` sh
#!/bin/sh
```

On Unix-like OS’s, the `#!` tells the program loader that this is a script to be interpreted, and `/bin/sh` is the path to the interpreter you want to use, `sh` in this case. Windows is definitely not a Unix-like OS. Git for Windows supports Bash commands and shell scripts via Cygwin. By default, what does it find when it looks for `sh.exe` at `/bin/sh`? Yup, nothing; nothing at all. Fix it by providing the path to the `sh` executable on your system. I’m using the 64-bit version of Git for Windows, so my shebang line looks like this.

``` shell
#!C:/Program\ Files/Git/usr/bin/sh.exe
```

### PreCommit GitHook to scan commit for keywords 
Let's go back to the example we started with, how could have GitHooks stopped Ryan Hellyer from accidentally leaking his Amazon AWS access keys to GitHub? You can invoke a script at pre-commit using GitHooks to scan the increment of code being committed into your local repository for specific keywords. Replace the code in this pre-commit shell file with the below code... 

``` sh
#!C:/Program\ Files/Git/usr/bin/sh.exe
matches=$(git diff-index --patch HEAD | grep '^+' | grep -Pi 'password|keyword2|keyword3')
if [ ! -z "$matches" ]
then
    cat <<\EOT
Error: Words from the blacklist were present in the diff:
EOT
    echo $matches
    exit 1  
fi
```

> Of course, you don't have to build the full key word scan list in this script, you can branch off to a different file by referring it here that you could simply encrypt or scramble if you wanted to...

Watch the video below on how you can do this for a Git repository in VSTS on Windows (Start the video at 2:10 to directly jump into this example)... 


{% include youtubeplayer.html id='8MqVc9mbrDk?t=127' %}

``` PowerShell













```

The `repo\.git\hooks` folder is not committed into source control, so you may ask how do you share the goodness of the automated scripts you create with the team? The good news is that from Git version 2.9 you now have the ability to map githooks to a folder that can be committed into source control, you could do that by simply updating the global settings configuration for your git repository... 

``` shell

git config --global core.hooksPath '~/.githooks'

```

If you ever need to overwrite the GitHooks you have set up on the client side, you could do so by using the no-verify switch...

``` shell

git commit --no-verify 

```

### Server side GitHooks with VSTS 
So far we have looked at the client side GitHooks on Windows, VSTS also exposes server side hooks. VSTS uses the same mechanism itself to create Pull requests... 

![Server side GitHooks in VSTS](/images/screenshots/tarun/GitHooksVSTSServerSide.PNG)


I'll show you some cool stuff with side hooks in VSTS in a future post, for now you can read up more about it [here](https://www.visualstudio.com/en-us/docs/integrate/get-started/service-hooks/events#tfvc.checkin)

Hope you found this useful, stay tuned for more cool stuff with Git!

Tarun   