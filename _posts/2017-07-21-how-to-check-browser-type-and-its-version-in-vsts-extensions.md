---
layout: post
title: "HOWTO: Check browser type and its version in your VSTS/TFS extensions"
date: 2017-07-21
author: utkarsh
tags: ["Extensions", "VSTS"]
categories:
- "Extensions"
- "VSTS"
img: "/images/screenshots/utkarsh/Jul17/browser-vsts.png"
description: "How to copy text to clipboard in VSTS extensions using VSTS Web Extension SDK"
permalink: extensions/how-to-copy-text-to-clipboard-in-vsts-extensions
keywords: "Extensions, VSTS"
---

Ever wanted to check which browser (and browser version) your VSTS/TFS extension is running on? Microsoft Visual Studio Team Services [Web Extension SDK](https://github.com/Microsoft/vss-web-extension-sdk) has a great set of utility methods. In this blog post we will see couple of such methods which will help us to detect browser type and its version.

<!--more-->

![Image](/images/screenshots/utkarsh/Jul17/browser-vsts.png)


## Steps ##


Like in previous post we need to import the module in to our typescript file. The methods we are interested are all in `UI` module.

```ts
import BrowserCheck = require("VSS/Utils/UI");
```

The imported UI module provides various methods to check the browser and its properties.


### Check for Google Chrome ###

```ts
let chrome = BrowserCheck.BrowserCheckUtils.isChrome();
```

### Check for Edge browser ###

```ts
let edge = BrowserCheck.BrowserCheckUtils.isEdge();
```

### Check for Firefox, Safari and IE ###

Similar to the methods above, you can check other browser types too.

```ts
let firefox = BrowserCheck.BrowserCheckUtils.isFirefox();
let safari = BrowserCheck.BrowserCheckUtils.isSafari();
let ie = BrowserCheck.BrowserCheckUtils.isIE();
```

### Check for browser version ###

You can get the browser version like below.

```ts
let version = BrowserCheck.BrowserCheckUtils.getVersion();
```

### Check for specific IE version ###

If you are looking to check specific version of IE, there is method for it too. 

```ts
let isIE11 = BrowserCheck.BrowserCheckUtils.isIEVersion(11);
```

### Check if browser version is less than v8 or v9 ###

Now if you would like to check if extension is running on lower version than IE8 or IE9, the SDK has methods to check exactly that.

```ts
let lessThanIE8 = BrowserCheck.BrowserCheckUtils.isLessThanOrEqualToIE8();
let lessThanIE9 = BrowserCheck.BrowserCheckUtils.isLessThanOrEqualToIE9();
```

### Can I detect if my extension is running on Windows/Macintosh/iOS? ###

Yes, you can!

```typescript
let isWindows = BrowserCheck.BrowserCheckUtils.isWindows();
let isMac = BrowserCheck.BrowserCheckUtils.isMacintosh();
let isiOS = BrowserCheck.BrowserCheckUtils.isIOS();
```

That's it for this post. Thanks for reading!

