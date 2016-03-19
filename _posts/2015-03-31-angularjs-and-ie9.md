---
layout: post          #important: don't change this
title: "AngularJS and Internet Explorer 9"
date: 2015-03-31 05:15:00 
author: Utkarsh Shigihalli
categories:
- blog                #important: leave this here
- angularjs
- mvc
- IE
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---

[AngularJS](https://angularjs.org/) is a popular web framework to build interactive web pages. It is compatible with many available modern browsers like Chrome, Firefox and Internet Explorer (IE). However, AngularJS and older version IE (v6 to v9) does not work very well and you need to perform few additional actions for supporting IE. AngularJS already has a separate page and provides guidance to support IE browser [here](https://docs.angularjs.org/guide/ie).
 
However, there are other issues as well which are not particulary related to AngularJS and some are related to how IE browser (<= version 9) works. I will try to cover few of such issues in this blog post.
<!--more-->

### 1. Data fetched only for the first time or Requests are not sent to the server ###

**CAUSE:** 

Internet Explorer by default caches AJAX requests from the server. Hence, the first request is fetched from the server, but subsequent requests are served from the cached version. Hence, you will not get the latest data.

**SOLUTION 1 - Randomize the URL :**

If you are getting the list of users by making a request to `/api/getusers` next request to the same url is served from the cached results and not from the server. To avoid this, you can randomize this url, so that IE thinks as a unique requst. Simplest way to do this is by sending a random parameter with timestap appended. This way every request is unique and IE will query the server each time.

```js
//pass the timestamp as rnd query parameter 
$http.get('/api/getusers?rnd='+ new Date().getTime() }).
  success(function(data, status, headers, config) {
    // this callback will be called asynchronously
    // when the response is available
  }).
  error(function(data, status, headers, config) {
    // called asynchronously if an error occurs
    // or server returns response with an error status.
  });
```

**SOLUTION 2 - Change HTTP requests to POST instead of GET:**

You can convert all your GET requests to POST. By default, http POST requests are never cached by browser and hence every request goes to the server. Read more on [GET vs POST](http://www.w3schools.com/tags/ref_httpmethods.asp)

```js
$http.post('/api/getusers').
  success(function(data, status, headers, config) {
    // this callback will be called asynchronously
    // when the response is available
  }).
  error(function(data, status, headers, config) {
    // called asynchronously if an error occurs
    // or server returns response with an error status.
  });
```

**SOLUTION 3 - Disable Caching in your application:**

If you are working on ASP.NET MVC application, you can disable the caching for your application. This can be done in Global.asax.cs file as below.

```csharp
protected void Application_PreSendRequestHeaders(object sender, EventArgs e)
{
    Response.Cache.SetCacheability(HttpCacheability.NoCache);
}
```

Alternatively, If you dont want to disable caching globally for your application, you can disable for individual actions as in below code

```csharp
[OutputCache(NoStore = true, Duration = 0, VaryByParam = "None")]
public ActionResult GetUsers()
{
    // return your response
}
```

**SOLUTION 4 - Disable Caching in AngularJS:**

I got this solution from Stackoverflow and this is the **best option** in my view. You can disable caching for all your requests in AngularJS application by importing [$httpProivder](https://docs.angularjs.org/api/ng/provider/$httpProvider) and configure not to have cache. 

```js
myModule.config(['$httpProvider', function($httpProvider) {
//initialize get if not there
if (!$httpProvider.defaults.headers.get) {
	$httpProvider.defaults.headers.get = {};    
}    
//disable IE ajax request caching
$httpProvider.defaults.headers.get['If-Modified-Since'] = 'Mon, 26 Jul 1997 05:00:00 GMT';
$httpProvider.defaults.headers.get['Cache-Control'] = 'no-cache';
$httpProvider.defaults.headers.get['Pragma'] = 'no-cache';
}]);
```

### 2. AngularJS does not execute or executes only when IE developer tools are enabled ###

**CAUSE**

Because you have `console.log` or `console.time` in your code. Surprised? but it is true. IE does not recognize console.* messages by default. They are recognized only when IE Developer Tools are enabled and hence AngularJS is executed only when Developer Tools are enabled. I use [console.time](https://developer.chrome.com/devtools/docs/console-api#consoletimelabel) and [console.timeEnd](https://developer.chrome.com/devtools/docs/console-api#consoletimeendlabel) methods in my JS files, to analyze the performance of my AngularJS methods. I also had lot of `console.log` messages scattered all accross my JS files which caused the IE to error out and not execute AngularJS code. 

**SOLUTION 1: Disable console.* messages using Javascript**

Although you can remove console messages from JS files, it is not an ideal approach as rest of the browsers handle them gracefully. So to disable these messages, you can place following javascript function (found in [GitHub](https://github.com/h5bp/html5-boilerplate/blob/master/src/js/plugins.js)) in your html page or in my case _Layout page (in MVC).
The following code block disables all the `console.*` messages if console object is not initliazed. This ensures IE not to error out and hence Angular JS is executed properly.

```js
// Avoid `console` errors in browsers that lack a console.
(function() {
    var method;
    var noop = function () {};
    var methods = [
        'assert', 'clear', 'count', 'debug', 'dir', 'dirxml', 'error',
        'exception', 'group', 'groupCollapsed', 'groupEnd', 'info', 'log',
        'markTimeline', 'profile', 'profileEnd', 'table', 'time', 'timeEnd',
        'timeline', 'timelineEnd', 'timeStamp', 'trace', 'warn'
    ];
    var length = methods.length;
    var console = (window.console = window.console || {});

    while (length--) {
        method = methods[length];

        // Only stub undefined methods.
        if (!console[method]) {
            console[method] = noop;
        }
    }
}());
```

## 3. AngularJS expressions are briefly displayed while page is loading###

**CAUSE**

AngularJS expressions are evaluated when DOM is completely loaded and upon evaluation of the expression the value is replaced. However, sometimes you see that there is a flicker effect where the expressions are displayed for few seconds and then replaced with actual value. This becomes bit annoying after some time.

**SOLUTION 1: Replace expressions with ng-bind**
Angular expressions like `{{model.userName}}` can be replaced with ng-bind attribute. 

```html
<!--Instead of this-->
<span>{% raw %}{{model.userName}}{% endraw %}</span>
<!--use ng-bind-->
<span ng-bind="model.userName"></span>
```

This ensures there are no expressions on the markup and hence are not visible still AngularJS is compiled.

**SOLUTION 2: Use AngularJS ng-cloak**

ng-cloak [documentation](https://docs.angularjs.org/api/ng/directive/ngCloak) explains it well.

> The ngCloak directive is used to prevent the Angular html template from being briefly displayed by the browser in its raw (uncompiled) form while your application is loading. Use this directive to avoid the undesirable flicker effect caused by the html template display.

To use this, you can decorate `body` tag in your HTML with `ng-cloak` attribute. This hides the expressions while they are evaluated and hence you will not see the flicker effect.

For better working (and to support older browsers) you may also want to add the below CSS class to your CSS file.

```cs
[ng\:cloak], [ng-cloak], [data-ng-cloak], [x-ng-cloak], .ng-cloak, .x-ng-cloak {
  display: none !important;
}
```

The ng-cloak attribute is automatically deleted by AngularJS once the expressions are evaluated, hence making the elements visible.