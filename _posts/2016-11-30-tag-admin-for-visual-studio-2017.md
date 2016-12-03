---
layout: post
title: "Tag Admin extension is now available for Visual Studio 2017"
date: 2016-12-03
author: utkarsh 
tags: ["VisualStudio", "Extensions"]
categories:
- "Extensions"
img: "/images/screenshots/utkarsh/tagadmin/main.png"
description: "Tag Admin for Visual Studio 2017"
permalink:  
keywords: "Extensions"
---

Tarun and I, developed [Tag Admin](https://marketplace.visualstudio.com/items?itemName=UtkarshShigihalliandTarunArora.TagAdminforVisualStudio2015)  extension for Visual Studio around [2 years](http://www.visualstudiogeeks.com/blog/tagadmin/visualstudio-tags-administration-using-extension-tagadmin) back. Over a period of time, we got busy and the extension has developed few bugs, mostly because there were few  changes in the REST API and we could not update the extension accordingly. 

However, now that Visual Studio 2017 [is out](https://www.visualstudio.com/vs/visual-studio-2017-rc/), it was time to update the extension and make it compatible with Visual Studio 2017 and also fix the bugs :-). 

<!--more--> 

![Tag Admin for Visual Studio 2017](/images/screenshots/utkarsh/tagadmin/main.png)

> Download the extension from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=UtkarshShigihalliandTarunArora.TagAdminforVisualStudio2017). If you are keen to try out 'under development' bits, please download the VSIX directly from [CI build](http://vsixgallery.com/extension/96554fd6-649e-46f9-9162-3291177d9379/)

## Upgrade process ##

The whole upgrade process was easy. Visual Studio 2017 is super light weight and has a new VSIX extension manifest [format](https://docs.microsoft.com/en-us/visualstudio/extensibility/what-s-new-in-the-visual-studio-2017-sdk) (VSIX3). There is also a step-by-step [guide](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-migrate-extensibility-projects-to-visual-studio-2017) on how to migrate extensions to Visual Studio 2017. 

I just had to follow the guide and my extension was ready to be used with Visual Studio 2017.

## Fixing existing bugs in the extension ##

Our Visual Studio 2013 and 2015 extensions used the [REST API](https://www.visualstudio.com/en-us/docs/integrate/api/wit/tags) and we used to construct the URL's required as per the API. 

For Visual Studio 2017 extension though, I decided to use the new [REST based HTTP client](https://www.visualstudio.com/en-us/docs/integrate/get-started/client-libraries/samples) which made development easy. You no longer required to construct the URL's. Most of the REST calls happens behind the scene.

Take for example the below code snippet to `Get the Tags`. 

Old code with REST URL

```cs
var restRequest = includeInActive ? string.Format("_apis/tagging/scopes/{0}/tags?includeInactive=true", Scope) : string.Format("_apis/tagging/scopes/{0}/tags", Scope);
var request = new HttpRequestMessage();
AddReqestHeaders(request);
request.Method = HttpMethod.Get;
request.RequestUri = _baseUrl.AddRestParameter(restRequest);
var response = await SendAsync(request);
tagResponse.Data = JsonConvert.DeserializeObject<T>(response.Content.ReadAsStringAsync().Result);
return tagResponse.Data;
```

New code with REST HTTP client

```cs
var tagClient = await GetTagClient();
var tags = await tagClient.GetTagsAsync(Scope);
return tags;
```

As you can see, the code is simple and also readable. So converting existing code to use new REST HTTP client went super smooth and it was easier that I expected.

## Download ##

So, please go ahead and try out the [new version](https://marketplace.visualstudio.com/items?itemName=UtkarshShigihalliandTarunArora.TagAdminforVisualStudio2017) and let me know if you have any feedback.