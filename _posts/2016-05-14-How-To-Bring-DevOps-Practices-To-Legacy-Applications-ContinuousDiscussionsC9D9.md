---
layout: post
title: "How to bring DevOps practices to legacy applications (#c9d9 - ContinuousDiscussions)"
date: 2016-05-14 20:30:00 
author: tarun 
tags: ["DevOps", "Agile"]
categories:
- blog                #important: leave this here
- "DevOps"
permalink:  /blog/DevOps/How-To-Bring-DevOps-Practices-To-Legacy-Applications-c9d9
description: "DevOps LegacyApplications ContinuousDiscussions c9d9"
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /images/screenshots/thumbs/
---
Recently I participated in an online panel to discuss how best to tackle Continuous Delivery for Legacy Applications. This was one in the series of Continuous Discussions (#c9d9). #c9d9 is a series of community panels about _Agile_, _Continuous Delivery_ and _DevOps_. Read on for some of the key points that emerged in this discussion... Feel free to leave a comment if you have any suggestions of your own...     
<!--more--> 

![c9d9 - Tarun in action - DevOps](/images/screenshots/tarun/c9d9-LegacyDevOps.png)

> __Legacy?__ Assuming your business did not start yesterday – you probably have some of those lying around: those applications that might be poorly understood, or poorly tested, that can be cumbersome or brittle, that may be difficult to update — but that somehow – still – just work. This is the legacy code that keeps the lights on, but that you’re afraid to touch…
 
![What is Legacy - DevOps](/images/screenshots/tarun/c9d9-DevOpsWhatIsLegacy.gif)

# I like it, where can I watch it? 
Glad you ask, you can watch the full video on youtube...  

<iframe width="560" height="315" src="https://www.youtube.com/embed/VSDzsZRZyLs" frameborder="0" allowfullscreen></iframe>

This episode features:

__Deepak Karanth__
Chief Architect and Consultant. Agile and DevOps Champion. Deepak also helps companies with technical strategy and process improvements.

@SoftwareYoga [softwareyoga.com](http://softwareyoga.com/)

__Jean D'Amore__
Consultant at ThoughtWorks Australia and corsican polyglot programmer who likes to be underwater, cook, and keep his garden green.

@jeandamore [blog.corsamore.com](https://blog.corsamore.com/)

__Manuel Paid__
Dev+Build+QA=#DevOps advocate. People-first technologist @ Skelton-Thatcher. Jack of all trades, master of continuous improvement.

@manupaisable [infoq.com/author/Manuel-Pais](http://www.infoq.com/author/Manuel-Pais)

__Tarun Arora__
Tarun is obsessed with high-quality working software, continuous delivery and Agile. He is a Microsoft MVP in Visual Studio Development Tools and the author of ‘DevOps & ALM with TFS 2015’.

@arora_tarun [visualstudiogeeks.com](http://www.visualstudiogeeks.com/)

_'__Credits__ - Continuous Discussions is a community initiative by [Electric Cloud](http://electric-cloud.com/powering-continuous-delivery), which powers Continuous Delivery at businesses like SpaceX, Cisco, GE and E*TRADE by automating their build, test and deployment processes.'_

Read on for a summary of my thoughts on the subject below... Insights that hopefully you find useful as you make your journey to DevOpsify some of the legacy in your enterprise...  

# How do you define legacy?
Legacy application in my opinion is not the end state, it's usually the first. And it starts with the fear of changing the application and that is usually a decision management would take because they so truthfully respect stability in their core business set of applications, that the fear of it going down becomes so big that they are just afraid of modifying it. And that's where the legacy starts creeping in and it seems to move on and nobody knows about the application, the code becomes stale, there is no continuous improvement, there is no continuous deployment, it becomes an application of pre-DevOps because no one understands the DevOps process around it, not to say that it never had a DevOps process. So it's not all code, it's not just having a unit test, it's entirely about the ability to change it on demand, if you can do that you are already on the path to legacy.


# When should you poke the bear?
I think you have to take a business case driven approach here, you may have legacy applications but you may not have a business value chain in that area anymore to invest in revamping that application or rewriting it. So it's important to go through the enterprise to figure out what takes stocks, what applications you have, what is the turnaround time on them and correct them based on the value they deliver to your organization. That probably is your decision point as to whether it's even worth it in the first place. Having said that, it's about having a good tool-chain as well, if you can't track things like technical data in your application then you are relying on word of mouth about whether the application is legacy or not.


# What are challenges with CI/CD for legacy applications?
For me it's about emphasizing speed, about comprising policy. We all have finite resources and if I'll invest in improving the CD/CI, I need to know where the biggest impact area is. I'll take an example of an application I worked on recently, the app was written in Power Builder which is way out of date now. Finding someone with that skillset, good luck with that, to instrument the application with a package we bought, and track the usage of the app to know what the hot spots in the application are. We would invest some time and effort to make sure we have some unit tests or automated testing to have us turn the handle quicker and where to we make that investment. Users were telling us that they don't use this part of the application, yet the data was telling us that they do, only to find out that there are other departments also using the application and it was nowhere to be tracked. These surprises will come along. But for me the approach is to know where you're going to get the biggest bang for your buck, and that's where the investment needs to go in. Another example is an off-the-shelf application I worked on recently, Endure OpenLink. It's a packaged application so you don't have an SDK or an API you can invoke through automation. And that's where you rely on the expertise of developers or other people who have worked on it. Sometimes you have to scavenge in forums to read up and see what hidden endpoints are valuable. But it's all about decomposing it in a fashion where either you create a facade around it, or you identify the endpoint that you can use to do some sort of test automation, or some sort of automation to speed up the CI/CD pipeline. 

# Culture? Tools? Metrics? Where do you start!
To me, there needs to be business value, appetite for change, and a roadmap and vision. If you lack any of them, let it go and poke the bear. There is no value in doing it if there's no value in doing it.

# Summary 
You can find more insights on the subject on the electric cloud blog [here](http://electric-cloud.com/blog/2016/05/continuous-discussions-c9d9-podcast-episode-40-cd-legacy-applications/). 

How are you DevOps-ing Legacy applications? 

Tarun 