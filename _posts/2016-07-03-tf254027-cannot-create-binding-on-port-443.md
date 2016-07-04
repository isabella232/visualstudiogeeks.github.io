---
layout: post
title: "TF254027 : Cannot create a binding on port 443 because the specified certificate could not be found"
date: 2016-07-03
author: utkarsh 
tags: ["TFS2015", "DevOps", "TeamFoundationServer"]
categories:
categories:
- "TFS2015"
- "DevOps"
- "TeamFoundationServer"
img: "/images/screenshots/utkarsh/tf254027-cert-not-found/error.jpg"
description: "TF254027 : Cannot create a binding on port 443 because the specified certificate could not be found"
keywords: "TFS2015"
---

When doing the upgrade of TFS2015 from previous update version you might get an error as `TF254027 : Cannot create a binding on port 443 because the specified certificate could not be found`. This post will show you how you can solve the error and detailed steps to solve it.

<!--more-->

![Error](/images/screenshots/utkarsh/tf254027-cert-not-found/error.jpg)


## Cause ##

The error is caused if the certificate you used to configure HTTPS for TFS 2015 is not found.

## Solution ##

Click `Previous` button in the wizard till you go to `Application Tier` tab.

![ApplicationTier](/images/screenshots/utkarsh/tf254027-cert-not-found/app-tier-page.jpg)


Click `Edit Site Settings` to open the below dialog

![Edit Site Binding](/images/screenshots/utkarsh/tf254027-cert-not-found/website-settings.jpg)

Select `https` protocol (port 443) and click `Edit`. This will open a dialog below.

![Assign Certificate](/images/screenshots/utkarsh/tf254027-cert-not-found/edit-site-binding.jpg)

If you have your organization certificate, you should see it in the `SSL Certificate` dropdown. If you see the correct certificate, select it to use it - and you can skip rest of the blog post :-)

If you do not have any certificate (or certificate has been removed accidentally), you can create a self signed certificate using IIS. 

### Create a self signed certificate ###

To create self signed certificate you can follow the steps mentioned [here](http://weblogs.asp.net/scottgu/tip-trick-enabling-ssl-on-iis7-using-self-signed-certificates). 

### Assign the certificate ###

Once you create the self signed certificate, come back to the wizard and you should see the certificate.

![CertificateShown](/images/screenshots/utkarsh/tf254027-cert-not-found/edit-site-binding-cert.jpg)

Re-run the readiness check and you should see all checks pass.

![ReadinessComplete](/images/screenshots/utkarsh/tf254027-cert-not-found/success.jpg)



