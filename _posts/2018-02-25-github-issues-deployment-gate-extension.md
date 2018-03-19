---
layout: post
title: "GitHub Issues Release Gate extension for VSTS Release Management"
date: 2018-02-25
author: utkarsh
tags: ["DevOps", "VSTS", "GitHub", "ReleaseGates", "Extensions"]
categories:
- "DevOps"
description: "Do you want to stop deployment as long as there are outstanding issues in your GitHub repository? In this blog post we will see how we can leverage powerful VSTS deployment gate extension your next big deployment."
permalink: /DevOps/github-issues-deployment-gate-extension
keywords: "DevOps, VSTS, VSTS Deployment Gate Azure Function, Azure Function, Azure Function, GitHub, GitHub Issues, Deployment Gate, Release Management, Visual Studio, Extension"
image: "/images/screenshots/utkarsh/github-issues-deployment-gate/deployment-gate-success.png"
published: true
---

We saw in my [last blog post](https://www.visualstudiogeeks.com/DevOps/github-issues-as-deployment-gate-in-vsts-rm) how deployment gates allow you to monitor your GitHub repo for issues in your release pipeline. Microsoft recently made release gate extension point public for all users and we thought it would be a good idea to expose integration with GitHub repo (as discussed previously) as an VSTS extension. The advantage is that you do not have to worry about other infrastructure details such as writing and deploying the Azure function etc. Instead just install our extension from the link below, write the search string for finding out issues and you are all set!

<!--more-->

## Install the extension

This is super easy, head over to [extension page](https://marketplace.visualstudio.com/items?itemName=UtkarshShigihalliandTarunArora.github-issues-release-gate) in VSTS marketplace and install the extension. You should see our extension under release gates section in VSTS.

![Select Gate]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate-extension/gate-select.png)

## Using the gate

You can add gates either prior to the deployment or post deployment. Below is UI to select pre-deployment gate.

![Pre-Deployment]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate-extension/pre-deployment-step.png)

This will display all the gates available and select `Search GitHub issues` gate.

![Gate]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate-extension/gate-select-arrow.png)

## Configure and search for issues

You will see configuration options once you add this gate and it allows you to configure the search query and success criteria.

> Please use the GitHub compatible query. You can read more about search query [here](https://help.github.com/articles/searching-issues-and-pull-requests/)

![Configure]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate-extension/gate-configure.png)

## Feedback

If you have any questions or feedback do not hesitate to contact us at [@onlyutkarsh](https://twitter.com/onlyutkarsh) and [@arora_tarun](https://twitter.com/arora_tarun)