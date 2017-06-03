---
layout: post
title: "GitHooks with PowerShell on Windows to automate source control operations"
date: 2017-06-03
author: tarun
tags: ["DevOps", "GitHooks", "PowerShell"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/GitHooksWithPowerShellOnWindows.png"
description: "Heard about GitHooks and now wondering how you can apply PowerShell to it? Look no further this blogpost will show you how to leverage GitHooks on Windows with Visual Studio Team Services using PowerShell in your DevOps solution... "
permalink: /DevOps/UsingGitHooksWithVstsGitOnWindows
published: true
keywords: "DevOps, GitHooks, GitHooks on Windows, GitHooks with PowerShell, GitHooks with VSTS, PowerShell, SourceControl, VersionControl, GitAutomation, GitHooks PowerShell, Git client side Hooks, CD with Git"
---
There are a whole host of things one tends to check before committing code into the source control system. It's a proven fact that addressing technical issues in the product are less expensive the earlier they are identified. Git luckily gives you a bunch of events against your local repository that you can leverage to automate the pre commit checks in your codebase. The average windows user may find it difficult to script the actions using shell script. Luckily you can invoke PowerShell scripts on Windows for your GitHooks. In this blogpost we'll cover an end to end example... 
<!--more--> 

### Where are the GitHooks scripts? 
Git ships with a number of sample hook scripts, in case you didn't realise, these have always been here in your repository. Check out the .git folder in your repository...  `repo\.git\hooks`. These samples are disabled by default. For instance, if you open that folder you’ll find a file called `pre-commit.sample`. To enable it, just rename it to `pre-commit` by removing the `.sample` extension and make the script executable. When you attempt to commit using `git commit`, the script is found and executed. If your pre-commit script exits with a 0 (zero), you commit successfully, otherwise the commit fails. 

> Even this script will fail on Windows as it won't find the correct path to Shell executable. Check out how you can fix this [Using GitHooks Shell scripts with Visual Studio Team Services on Windows]({% UsingGitHooksWithVstsGitOnWindows %})

``` sh
#!/bin/sh
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".

if git rev-parse --verify HEAD >/dev/null 2>&1
then
	against=HEAD
else
	# Initial commit: diff against an empty tree object
	against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

# If you want to allow non-ASCII filenames set this variable to true.
allownonascii=$(git config --bool hooks.allownonascii)

# Redirect output to stderr.
exec 1>&2

# Cross platform projects tend to avoid non-ASCII filenames; prevent
# them from being added to the repository. We exploit the fact that the
# printable range starts at the space character and ends with tilde.
if [ "$allownonascii" != "true" ] &&
	# Note that the use of brackets around a tr range is ok here, (it's
	# even required, for portability to Solaris 10's /usr/bin/tr), since
	# the square bracket bytes happen to fall in the designated range.
	test $(git diff --cached --name-only --diff-filter=A -z $against |
	  LC_ALL=C tr -d '[ -~]\0' | wc -c) != 0
then
	cat <<\EOF
Error: Attempt to add a non-ASCII file name.

This can cause problems if you want to work with people on other platforms.

To be portable it is advisable to rename the file.

If you know what you are doing you can disable this check using:

  git config hooks.allownonascii true
EOF
	exit 1
fi

# If there are whitespace errors, print the offending file names and fail.
exec git diff-index --check --cached $against --
```

This is what the `pre-commit` script looks like, an average windows user may struggle with a shell script, luckily PowerShell scripts can be used as a substitute... 

Interested in learning how you can go from zero to DevOps; Learn real world strategies and application of DevOps. Learn how to use apply modern engineering practices with Azure & VSTS to go from Continuous Integration to Continuous Delivery to Continuous Deployment! 
[Open & Free course on DevOps - Ci to Cd]({% /DevOps/DevOpsTrainingCiCdWithGitVstsAzure %}) 

### Invoke PowerShell script in GitHook
Let's start off simply by removing everything in the pre-commit GitHook shell script and see how you can invoke a PowerShell script with GitHook in Git on a Windows machine. As you can see in the example below, we are simply just calling a PowerShell script from a shell script through a GitHook script. Ofcourse, you need to give the path to the PowerShell script, it's best to reference this path from the root of the repository. 

``` sh
#!C:/Program\ Files/Git/usr/bin/sh.exe
echo
exec powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\.git\hooks\AutoFix-VisualStudioFiles.ps1"
exit
```

### Automate Formatting Config files in a Git Repository using PowerShell GitHook
Alright with the basics of the plumbing out of the way, let's look at the example of doing something meaningful with this Pre-Commit GitHook. In this example the script scans the solution folder (the parent of the .git folder) for all app.config, web.config and *.csproj files and auto-formats them to minimize the possibility of getting merge conflicts based on the ordering of elements within these files.

> Most of the merge conflicts arise from reshuffled configuration files and project files, these are the most tricky conflicts to resolve. 

Here is what the script exactly does on each file type, 
- `app.config` & `web.config` files - sorts appSettings elements by key, in alphabetic order, sorts assemblyBinding.dependentAssembly elements alphabetically based on the assemblyIdentity.name attribute
- `.csproj` files - sorts appSettings elements by key, in alphabetic order, sorts Reference, ProjectReference & Compile elements

Did you know you can now use Pester to test your PowerShell script, see this example here on how to put Pester in action to unit test your PowerShell scripts and visualize the test results [Testing PowerShell with Pester and Visual Studio Team Services]({%/DevOps/TestingAzureAutomationPowerShellRunbooksWithPesterInTeamServices%})

``` powershell
Function AutoFix-WebConfig([string] $rootDirectory)
{
    $files = Get-ChildItem -Path $rootDirectory -Filter web.config -Recurse

    return Scan-ConfigFiles($files)
}

Function AutoFix-AppConfig([string] $rootDirectory)
{
    $files = Get-ChildItem -Path $rootDirectory -Filter app.config -Recurse

    return Scan-ConfigFiles($files)
}

Function Scan-ConfigFiles([System.IO.FileInfo[]] $files)
{
    $modifiedfiles = @()

    foreach($file in $files)
    {
        $original = [xml] (Get-Content $file.FullName)
        $workingCopy = $original.Clone()

        if ($workingCopy.configuration.appSettings -ne $null){
            $sorted = $workingCopy.configuration.appSettings.add | sort { [string]$_.key }
            $lastChild = $sorted[-1]
            $sorted[0..($sorted.Length-2)] | foreach {$workingCopy.configuration.appSettings.InsertBefore($_, $lastChild)} | Out-Null
        }

        if ($workingCopy.configuration.runtime.assemblyBinding -ne $null){
            $sorted = $workingCopy.configuration.runtime.assemblyBinding.dependentAssembly | sort { [string]$_.assemblyIdentity.name }
            $lastChild = $sorted[-1]
            $sorted[0..($sorted.Length-2)] | foreach {$workingCopy.configuration.runtime.assemblyBinding.InsertBefore($_,$lastChild)} | Out-Null
        }

        $differencesCount = (Compare-Object -ReferenceObject (Select-Xml -Xml $original -XPath "//*") -DifferenceObject (Select-Xml -Xml $workingCopy -XPath "//*")).Length

        if ($differencesCount -ne 0)
        {
            $workingCopy.Save($file.FullName) | Out-Null
            $modifiedfiles += $file.FullName
        }
    }

    return $modifiedfiles
}

Function AutoFix-CsProj([string] $rootDirectory)
{
    $files = Get-ChildItem -Path $rootDirectory -Filter *.csproj -Recurse
    $modifiedfiles = @()

    foreach($file in $files)
    {
        $original = [xml] (Get-Content $file.FullName)
        $workingCopy = $original.Clone()

        foreach($itemGroup in $workingCopy.Project.ItemGroup){

            # Sort the reference elements
            if ($itemGroup.Reference -ne $null){

                $sorted = $itemGroup.Reference | sort { [string]$_.Include }

                $itemGroup.RemoveAll() | Out-Null
 
                foreach($item in $sorted){
                    $itemGroup.AppendChild($item) | Out-Null
                }
            }

            # Sort the compile elements
            if ($itemGroup.Compile -ne $null){

                $sorted = $itemGroup.Compile | sort { [string]$_.Include }

                $itemGroup.RemoveAll() | Out-Null
 
                foreach($item in $sorted){
                    $itemGroup.AppendChild($item) | Out-Null
                }
            }

            # Sort the project references elements
            if ($itemGroup.ProjectReference -ne $null){

                $sorted = $itemGroup.ProjectReference | sort { [string]$_.Include }

                $itemGroup.RemoveAll() | Out-Null
 
                foreach($item in $sorted){
                    $itemGroup.AppendChild($item) | Out-Null
                }
            }
        }

        $differencesCount = (Compare-Object -ReferenceObject (Select-Xml -Xml $original -XPath "//*") -DifferenceObject (Select-Xml -Xml $workingCopy -XPath "//*")).Length

        if ($differencesCount -ne 0)
        {
            $workingCopy.Save($file.FullName) | Out-Null
            $modifiedfiles += $file.FullName
        }
    }

    return $modifiedfiles
}

$rootDirectory = Join-Path (Split-Path -Parent $MyInvocation.MyCommand.Path) "\..\..\"

$exitCode = 0;

$changedfiles = @()
$changedfiles += AutoFix-AppConfig($rootDirectory)
$changedfiles += AutoFix-CsProj($rootDirectory)
$changedfiles += AutoFix-WebConfig($rootDirectory)

if ($changedfiles.Count -gt 0)
{
    Write-Host "=== git hooks ==="
    Write-Host "The following files have been auto-formatted"
    Write-Host "to reduce the likelyhood of merge conflicts:"
    
    foreach($file in $changedfiles)
    {
        Write-Host $file
    }

    $exitCode = 1;
}

exit $exitcode
```

Save this script in the folder `.git\hooks\` with the name `AutoFix-VisualStudioFiles.ps1`. 

Did you know... Ryan Hellyer accidentally leaked his Amazon AWS access keys to GitHub and woke up to a $6,000 bill the next morning. Wouldn't you just expect the source control as clever as git to just stop you from making such a blender?! Well, in case you didn't know you could put Git Hooks to work to address not just this but many similar scenarios... [Scan my pre-commits using GitHook to detect any password using a keyword builder]({%/DevOps/UsingGitHooksWithVstsGitOnWindows%})

In order to put in action, simply make some changes in your repository and commit the code. This will invoke the pre-commit event which will intern invoke this script. The result, your configuration files and csproj files will be organized. Watch the video below on how you can do this working for a Git repository in VSTS on Windows (Start the video at 8:45 to directly jump into this example of doing this with PowerShell)... 


{% include youtubeplayer.html id='8MqVc9mbrDk?t=127' %}

``` PowerShell













```

Hope you found this useful, stay tuned for more cool stuff with Git!

Tarun   
