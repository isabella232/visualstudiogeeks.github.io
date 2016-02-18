---
layout: post          #important: don't change this
title: "Walkthrough: Scheduling jobs using Quartz.net – Part 1: What is Quartz.Net?"
date: 2015-01-04 22:12:00
author: Tarun Arora
categories:
- blog                #important: leave this here
- "Quartz.Net"
img:        #place image (850x450) with this name in /assets/img/blog/
thumb: thumb-icon-tarun.jpg    #place thumbnail (70x70) with this name in /assets/img/blog/thumbs/
---
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-56c6503fb913a4a1"></script>
[Quartz.NET](http://www.quartz-scheduler.net/) is a full-featured, open source enterprise job scheduling system written in .NET platform that can be used from smallest apps to large scale enterprise systems. I want to schedule the execution of a task but only when something happens. Let’s call that something a trigger, so... if the trigger is met => execute the task. 
<!--more-->

![If Then Else](/assets/img/blog/tarun/post06_QuartzPost01IfThenElse.jpg). 

Sounds simple, why not use windows task scheduler for this?


### What is the problem that we trying to address? ###
Well, windows task scheduler is great for tasks where the trigger can be easily defined. With windows task scheduler will you be able to schedule a task to run on every working day according to the UK calendar (exclude all weekends & bank holidays) without either writing the logic for day check in the task or a wrapper script calling into the task.

The task should just contain the execution logic and should not have anything to do with the schedule for execution; Quartz.net allows you to achieve this and lots more. A quartz.net trigger gives you the flexibility for task invocation based on the following triggers,

1. at a certain time of day (to the millisecond)
2. on certain days of the week
3. on certain days of the month
4. on certain days of the year
5. not on certain days listed within a registered Calendar (such as business holidays)
6. repeated a specific number of times
7. repeated until a specific time/date
8. repeated indefinitely
9. repeated with a delay interval

Did 8 – repeat indefinitely just ring a bell? I’ll be covering that in the future post.

### Using Quartz.net as a windows service ###
You can have Quartz.net run as a standalone instance within its own .NET virtual machine instance via .NET Remoting. Let’s take a look at typical application architecture. In the figure below, I have the application tier set up on Machine 1, database set up on Machine 2 and Quartz.net set up on Machine 3 which is normally the architecture for most (if not all) enterprise applications.

![Figure 1 - Typical Application architecture while using Quartz.net as a windows service](/assets/img/blog/tarun/post06_QuartzPost01LogicalArchitecture.jpg)

###### Figure 1 -  Typical Application architecture while using Quartz.net as a windows service ######

### What other options do I have if I don’t want to use Quartz.net? ###
Quartz.net is just one of the many job scheduling services. Have a look at this comprehensive list of free and paid enterprise job scheduling software along with their feature comparison. [http://en.wikipedia.org/wiki/List_of_job_scheduler_software](http://en.wikipedia.org/wiki/List_of_job_scheduler_software)

This was first in the series of posts on enterprise scheduling using Quartz.net, in the next post I’ll be covering how to Install Quartz.net as a windows service. All Quartz.Net specific blog posts can listed [here](http://www.visualstudiogeeks.com/category/#Quartz.Net). Thank you for taking the time out and reading this blog post. If you enjoyed the post, remember to <a href="http://feeds.feedburner.com/visualstudiogeeks/otas" rel="alternate" type="application/rss+xml"><img src="//feedburner.google.com/fb/images/pub/feed-icon16x16.png" alt="" style="vertical-align:middle;border:0"/>Subscribe to feed</a>. Stay tuned!