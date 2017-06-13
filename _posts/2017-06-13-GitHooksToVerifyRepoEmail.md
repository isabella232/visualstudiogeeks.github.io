---
layout: post
title: "DevOps: GitHook to verify repository email address"
date: 2017-06-13
author: tarun
tags: ["DevOps", "Git", "GitHooks"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/ClientSideGitHooksGitWindowsVSTS.PNG"
description: "Use a PreCommit GitHook to validate the email address configured in the configuration to avoid accidentally committing changes in work repository with personal email and visa versa..."
permalink: /DevOps/GitHookToVerifyRepositoryEmailAddressOnWindows
published: true
keywords: "DevOps, GitHooks, PowerShell,  Git, PreCommit, Repository, GlobalConfiguration, GitHooks Windows, Global GitHooks, Git Conditional Includes, GitHooks Tarun"
---
When you are using Git for work and personal repositories although you can logically separate them in separate folders, you can accidentally commit in your personal repository with your work account and visa versa. It is annoying and in some cases might break employer policies as well. Luckily with GitHooks it's possible to set up a pre-commit hook that allows you to use the correct email address for the correct repository, check the blogpost for how... 
<!--more--> 

# Scenario 
So, let's work through this scenario... When you set up your Git repository for the first time you'll set up a personal username and email globally with the following commands...

``` powershell
git config --global user.name "Tarun Arora"
git config --global user.email "tarun_dot_arora@outlook.com"
```

You can obviously overwrite this per git repository as well by running the following commands within the scope of the specific repository...

``` powershell
cd myWorkRepository
git config user.email "tarun_dot_b_dot_arora@avanade.com"
# print the configuration
git config user.email
tarun_dot_b_dot_arora@avanade.com
``` 

Now you would probably clone your work repositories hypothetically in the directory `C:\Repos\Work\` and clone your personal repositories hypothetically in the directory `C:\Repos\Personal\` ...

Now if you __forget__ to set up an email for your work repository, your globally configured personal email address would be used when committing to your work id. Not ideal! 


Interested in learning how you can go from zero to DevOps; Learn real world strategies and application of DevOps. Learn how to use apply modern engineering practices with Azure & VSTS to go from Continuous Integration to Continuous Delivery to Continuous Deployment! 
[Open & Free course on DevOps - Ci to Cd](http://www.visualstudiogeeks.com/DevOps/DevOpsTrainingCiCdWithGitVstsAzure)


# GitHooks to the rescue 
Git hooks allow you to run custom scripts whenever certain important events occur in the Git life-cycle, such as committing, merging, and pushing. Git ships with a number of sample hook scripts in the `repo\.git\hooks` directory. Read up my other post on a quick whip on GitHooks [here](http://www.visualstudiogeeks.com/DevOps/UsingGitHooksWithVstsGitOnWindows#so-where-do-i-start)
 
More specifically it is the Pre-Commit GitHook that we'll put to work to trigger a script ahead of committing the changes to validate the email address...

If you are using Windows, be sure you check out this quick note on using GitHooks on Windows in my other blog post [here](http://www.visualstudiogeeks.com/DevOps/UsingGitHooksWithVstsGitOnWindows#githooks-oh-windows)

``` sh
#!C:/Program\ Files/Git/usr/bin/sh.exe
PWD=`pwd`
if [[ $PWD == *"Work"* ]] # 1)
then
  EMAIL=$(git config user.email)
  if [[ $EMAIL == *"avanade"* ]] # 2)
  then
    echo "";
  else
    echo "email not configured to Avanade in Avanade directory";
    echo "run:"
    echo '   git config user.email "tarun_dot_b_dot_arora@avanade.com"'
    echo ''
    exit 1; # 3)
  fi;
fi;

```

If shell isn't your think, luckily GitHooks support PowerShell as well, check out my other blogpost on how to get GitHooks working on Windows with PowerShell [here](http://www.visualstudiogeeks.com/DevOps/UsingPowerShellForGitHooksWithVstsGitOnWindows#invoke-powershell-script-in-githook) 

So, let's see what we did here, 
+ Check the directory the repository resides in
+ If the repository is in the work directory 
+ Check the email currently configured
+ If it's not Avanade, set it up to be Avanade  

# What are the other options? 

Since Version 2.9 of Git, there is now built in support for global GitHooks that you can share by committing with in the repository... This means, not can you only benefit from this GitHook, but a generic version of this can be put to work for others that clone this repository...

Now version 2.13 of GIt, supports conditional includes... See Below...

``` cmd
 * The configuration file learned a new "includeIf.<condition>.path"
   that includes the contents of the given path only when the
   condition holds.  This allows you to say "include this work-related
   bit only in the repositories under my ~/work/ directory".
```

Think of it as a global configuration that gives you an option apply the global configuration depending on specific conditions, i think you have got the idea... Check out this [blogpost](https://www.edwardthomson.com/blog/git_conditional_includes.html) from Thomas Edward that covers the implementation on how to set this up...

Hope you found this useful...

Tarun 