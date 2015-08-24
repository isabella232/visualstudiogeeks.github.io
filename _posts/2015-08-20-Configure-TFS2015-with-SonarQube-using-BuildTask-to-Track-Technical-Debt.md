---
layout: post          #important: don't change this
title: "Technical Debt - Why care? How to set up SonarQube to work with TFS using Build Task?"
date: 2015-08-20 17:16:00
author: Tarun Arora
categories:
- blog                #important: leave this here
- "SonarQube"
- "DevOps"
img:        #place image (850x450) with this name in /assets/img/blog/
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /assets/img/blog/thumbs/
---
What the heck is TechnicalDebt...? Why should I track it...? How do I track it in TFS...? How can I integrate SonarQube with TFS? Is there a build task I can use to instrument my code for Technical Debt as part of the build process? This blog post will hopefully answer some of these questions, or may be raise more questions...  <!--more-->

##The blog post is divided into 2 parts,
1. Why, How, Does it, Can I?
2. Integrate SonarQube with TFS 2015...? 

#PART 1
<hr/>

## What is Technical Debt?
Anyone who has written code professionally would know that at times you will find yourself using unrecommended techniques (ummm... hacks!) to get stuff done because of the timelines you have been asked to deliver in... No code reviews... Poor DevOps practices... Lack of unit testing... Too many tactical implementations... Not addressing undelying issues causing large number of bugs...  are major contributors to Technical Debt. Technical Debt doesn't hit you overnight, it's a slow and gradual process... Unlike Financial debt, technical debt is very hard to recognize. Technical Debt will slow your ability to deliver value. Are you seeing any of these signs? Are you that poor soul who has just taken over a legacy code base that has been poorly constructed & re-constructed overtime (time to ask for a payraise bob :)

## Why should you track TechnicalDebt?
 There is no right answer here... A high performing team may not need to track technical debt, developer would just fix issues as they find 'em. Other teams may find it useful to log issues as and when they find 'em. What ever side of the fence you stand, please spend less time trying to manage technical debt and more time trying to fix it.
 
 > But before you can start tracking TechnicalDebt.. You need to form your definition of Technical Debt... Order the list by some factor that works for your team, could be value, could be customer impact (chose a factor that works for your team)... 
 
 To avoid the 'treading water' situations... I've seen scrum teams have goals to keep the technical debt under 'x'. Scrum teams will drop what they are doing and start paying off technical debt to bring it under x. It may not be worth while paying off all of x, because believe it or not, the last 10 items in x could be very low value. 
 
## How do I track technical debt?  
I am expecting the web traffic to the page to drop off by this point... Thank you if you are still reading... [***SonarQube***](http://www.sonarqube.org/, "SonarQube") is an open source platform that is the de facto solution for understanding and managing technical debt. There are other tools available in the market, but for the purposes of this post, we'll stick to SonarQube. 

## Does SonarQube have a C# plugin? 
Although SonarQube had a C# plugin before, with the new set of components it becomes really easy to share the following data:

* results of .Net and JavaScript code analysis
* code clone analysis
* code coverage data from tests
* metrics for .Net and JavaScript

In addition, SonarSource (the company behind SonarQube) have produced a set of .Net rules, written using the new Roslyn-based code analysis framework, and published them in two forms: a nuget package and a VSIX. This makes it possible to run the same set of rules in SonarQube AND directly in Visual Studio.

> Here is the official link to the announcement announcing the integration with MSBuild <a> http://www.sonarqube.org/announcing-sonarqube-integration-with-msbuild-and-team-build/</a> 

## PART 2
<hr/>

## How can I integrate SonarQube with TFS 2015?
The ALM Rangers have created an installation & set up guidance for configuring SonarQube to work with SQL Sever. The guidance is more like a living document, that can be previewed in the gitRepository [here](https://github.com/SonarSource/sonar-.net-documentation/blob/master/doc/installation-and-configuration.md). I won't duplicate the documentation here but call out a few things I tripped over...

> The log file sonar.log is created in the folder .\SonarQube\sonarqube-5.1.x\logs. If you can't get SonarQube configuration to work make sure you look into the log. It gives you very description error messages to work with...

### Got-ya's  
1. You won't be able to get SonarQube to work without the pre-requisties specified in the guide, make sure you follow the instructions in the guide to set up the [pre-requisites](https://github.com/SonarSource/sonar-.net-documentation)... 
	+ Java SE Runtime  
	+ Supported SQL Server drivers  
	+ SQL Server connection string 
2. In your SQL Server connection string don't specify the instance name as '.\SQLExpress', instead use '.\\\SQLServer', the .\ would be parsed to . which would make the instance name syntactially incorrect. Refer to the [Build SQL URL samples](https://msdn.microsoft.com/en-us/library/ms378428.aspx)  
3. Configure SQL Server to listen on a specific TCP Port, I used '1433' [How to configure SQL Server to listen on a specific port](https://msdn.microsoft.com/en-us/library/ms177440.aspx)  
4. Case sensitivity! You will run into the below exception if you don't specify the database name in the same case as it exists in the database. For example, if you specify 'Sonar' in the connection string the actual database name is 'sonar' you'll get the following exception. Use the same case!
 
> ActiveRecord::JDBCError: The database name component of the object qualifier must be the name of the current database. 

5. You don't have the Java SE Runtime installed or you don't have the PATH specified in environment variables... Navigate to the java jdk install location and run java.exe (default install location C:\Program Files\Java\jdk1.8.0_60\bin)  [How to fix this...](http://sonarqube-archive.15.x6.nabble.com/When-i-try-to-launch-sonar-for-the-first-time-on-my-XP-OS-installation-i-get-the-following-error-td3196666.html)

> Unable to execute Java command.  The system cannot find the file specified. (0x2)

You may need to update the environment variables on the machine to the java.exe if you run into this problem. [Solution here](http://www.robertsindall.co.uk/blog/setting-java-home-variable-in-windows/) should help. 

> ERROR: JAVA_HOME not found in your environment, and no Java executable present in the PATH. Please set the JAVA_HOME variable in your environment to match the location of your Java installation, or add "java.exe" to the PATH. 

## Build Tasks configuration for SonarQube
To hook up the analysis to be run as part of the build process you need to invoke the MSBuild runner for SonarQube. This is what the build looks like after you have configured it using the steps specified in the [guide](https://github.com/SonarSource/sonar-.net-documentation/blob/master/doc/analyze-from-tfs.md)... In addition to following the guide, I've swapped the cmd runner with the powershell runner and used the [Dynamic Version Script](http://www.colinsalmcorner.com/post/build-vnext-and-sonarqube-runner-dynamic-version-script) provided by in ALM MVP Colin on his blog Colin's ALM Corner... 

<img src="/assets/img/blog/tarun/TechDebt01_BuildTasks01.png" alt="SonarQube Build Task" style="width:100%;height:100%"><sub><center><b>Image 1 - SonarQube: Build Task</b></center></sub>
	
If you run into any issues during the configuration, feel free to leave a comment in the blog, I will try my best to help...

## SonarQube integrated with TFS 2015 Build in Action...
Once the analysis has been completed, I can see the results in the sonarQube portal.

* The code base already has about 1 hour 12 minutes of technical debt 

<img src="/assets/img/blog/tarun/TechDebt01_SonarQubeTFS2015InAction.png" alt="SonarQube TFS2015 Analysis Results" style="width:100%;height:100%"><sub><center><b>Image 2 - SonarQube TFS 2015: Fabrikam Fiber Code Analysis</b></center></sub>
	
* The detailed analysis shows lines of code by class and function - drill down, the level of duplication, complexity in the code, issues categorized by severity and an overall complexity rating... 
<img src="/assets/img/blog/tarun/TechDebt01_SonarQubeTFS2015InAction02.png" alt="SonarQube TFS2015 Analysis Results 2" style="width:100%;height:100%"><sub><center><b>Image 3 - SonarQube TFS 2015: Fabrikam Fiber Code Detailed Analysis</b></center></sub>

<br />	
<hr />

## What More...
If you haven't already read through the content presented at the //Build/ conference as this new feature was announced...

<iframe src="https://onedrive.live.com/embed?cid=61D0A67D27B527D3&resid=61D0A67D27B527D3%21158402&authkey=AAspZn_gKnlsMnA&em=2" width="100%" height="327" frameborder="0" scrolling="no"></iframe> 


<hr />

## Summary
At the risk of upsetting those of you who came looking here for a silver bullet to magically detect and destroy Technical Debt, you’re going to be disappointed.

> It took work, inattention to detail, possibly poor craftsmanship, to build up the Technical Debt... It will take work, paying attention, and having professional craftsmen to pay off the Technical Debt. Good Luck!

Cheers,

Tarun