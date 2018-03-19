---
layout: post          #important: don't change this
title: "Using exceptionless to track exceptions in your VS Extension"
date: 2014-12-06 23:09:52
author: utkarsh
tags: [Extensions, VisualStudio]
categories:
- "extensions"
- "Visual Studio"
- "Tools"
description: "Using exceptionless to track exceptions in your VS Extension"
image:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
In the current competitive world, everyone knows releasing the product to market is only half the battle. You are expected to continuously monitor application for its usage, memory consumption, errors, to quickly address any issues in your application. Quickly addressing issues your customers facing is essential to keep existing customers with your product. There are already many solutions available in the market, which provide complete application monitoring and analytics for your application. Some popular ones are, [Microsoft Application Insights](http://www.visualstudio.com/en-us/explore/application-insights-vs.aspx) and [Telerik Analytics](http://www.telerik.com/analytics). However, if you are looking for tracking only exceptions in your applications and not interested in other usage trends, so that you quickly track unhandled exceptions, you have a friend in [exceptionless.com](https://exceptionless.com/)

![image]({{site.url}}/images/screenshots/utkarsh//2014_12_06_using_exceptionless_to_track_Image1.png "image")

**Few of its main features include:**

*   Real time error notification 
*   Group error messages 
*   Error trends and more   

### Integrating exceptionless

For this post, I will be integrating with VS Extension which I am working on. I am using WPF 

1. Create a exceptionless account and then create a project for your application.

2. Select the project type in the you are building app – I chose WPF for my extension, however exceptionless supports many projects like ASP.NET MVC, WebAPI, Windows Forms and even Console applications

3. Based on your application type, install the nuget package.

![image]({{site.url}}/images/screenshots/utkarsh//2014_12_06_using_exceptionless_to_track_Image2.png "image")

4. Get the app key from exceptionless and usually you need update your appKey in config file for any other type of applications. Because VS extensions emit the assembly and loaded by Visual Studio, you cannot use app.config. Instead you need to add appKey in AssemblyInfo.cs as below.

```cs
[assembly:Exceptionless("xxxxxxxxxxxxxxxxxxxxxxxxxxxxx")]
```

5. Finally, you need to register your exceptionless in the application initialization. For VS extension, it will be in `Initialize` method `<ExtensionName>Package.cs`

```cs
protected override void Initialize()
{
    base.Initialize();
    ExceptionlessClient.Current.Register();
    //Continue with your code
}
```
6. Once you configured, all unhandled exceptions are automatically reported. You can manually report your handled exceptions from anywhere in the code as below

```cs
try
{
    //code throwing exception
}
catch (Exception exception)
{
    exception.ToExceptionless().Submit();
}
```

You can also pass any other information (your custom object or add custom tags) with exceptions. 

```cs
try 
{
    throw new ApplicationException(Guid.NewGuid().ToString());
} 
catch (Exception ex) 
{
    ex.ToExceptionless()
        .AddObject(order, "Order", 
			excludedPropertyNames: new [] { "CreditCardNumber" }, 
			maxDepth: 2)
        .AddTags("Order", "User")
        .MarkAsCritical()
        .SetUserEmail(user.EmailAddress)
        .Submit();
}
```

That's it! You will receive your exceptions with full stack trace in real time with nice charts as below.

![image]({{site.url}}/images/screenshots/utkarsh//2014_12_06_using_exceptionless_to_track_Image3.png "image")

You can dig in inside individual exceptions to view full stack trace.

![image]({{site.url}}/images/screenshots/utkarsh//2014_12_06_using_exceptionless_to_track_Image4.png "image")

### Exceptionless is open source!

Exceptionless is completely open source and code is actively developed on [Github](https://github.com/exceptionless/Exceptionless). This has the Api and the the code for the exceptionless UI you saw in above screenshots. That means, **you can completely host it in your own web server and integrate inside your applications**. Having said that, you can simply use hosted exceptionless.com to quickly use it for your applications without setting up any infrastructure.

**Note:**

During debugging my VS extension, I kept on getting this error dialog in VS Experimental Instance. I have mailed them about the same, will update this post if I get any answer from them. Please see that this exception dialog was seen only in extensibility project type and rest other projects worked just fine.

![image]({{site.url}}/images/screenshots/utkarsh//2014_12_06_using_exceptionless_to_track_Image5.png "image")