---
layout: post
title: "Integrating VSTS Release Management with ServiceNow using Release Gates"
date: 2017-12-07
author: tarun
tags: ["DevOps", "VSTS", "ServiceNow", "ReleaseGates"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/Dec17/xxx.PNG"
description: ""
permalink: /DevOps/xxx
published: false
keywords: ""
---
Are you using ServiceNow for change management and VSTS for Release Management. Are you wondering how you could automate the release approval once a change has been approved for implementation in service now... Look no further, in this blogpost i'll show you how to leverage the *Release Gates* feature in VSTS to integrate with service now and hopefully help you remove a manual step from your release approval workflow...       
<!--more-->
VSTS allows you to create Release Gates... Read this official post for a low down on what [Release Gates](https://docs.microsoft.com/en-us/vsts/build-release/concepts/definitions/release/approvals/gates) do, to save you the trouble... Release Gates do exactly what they say...

> *Gate the release process*...  

Gates enable integration with a bunch of external services and API's that allow you to plug in anything from your 'live site monitoring tool' to your 'issue management tool' to your 'change management tool'... 

> #### Gates also allow **native** integration with _Azure functions_, which is what we'll use to build out our integration with Service Now.

# Service Now - API 
---
Now before you can integrate with ServiceNow, you need to know how to get data out of serviceNow. ServiceNow supports a proper [REST API](http://wiki.servicenow.com/index.php?title=REST_API). However, you need to be set up to use the REST API, this can be challenging in various enterprise organizations. Simply that or if you are stuck with a ServiceNow instance that's really old, you'll find it doesn't support the REST API.

I'll show you a little way to cheat, you don't really need to use the ServiceNow REST API to get data out of ServiceNow... ServiceNow stores everything in either purpose build tables or custom tables, which you can directly navigate to... 

 