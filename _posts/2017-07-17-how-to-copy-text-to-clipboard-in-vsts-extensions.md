---
layout: post
title: "HOWTO: Copy text to clipboard in VSTS/TFS Extensions"
date: 2017-07-19
author: utkarsh
tags: ["Extensions", "VSTS"]
categories:
- "Extensions"
- "VSTS"
img: "/images/screenshots/utkarsh/copy-to-clipboard-vsts-extension.png"
description: "How to copy text to clipboard in VSTS extensions using VSTS Web Extension SDK"
published: true
keywords: "Extensions, VSTS"
---

If you have used TFS/VSTS, you might have used "Copy to Clipboard" option at least few times - either on the workitem context menu or to copy the repository URL on *Clone Repository* dialog . In this blog post we will see how to implement that functionality and copy text to clipboard from your VSTS/TFS extensions using Microsoft Visual Studio Team Services [Web Extension SDK](https://github.com/Microsoft/vss-web-extension-sdk) and Typescript.

<!--more-->

![Image](/images/screenshots/utkarsh/copy-to-clipboard-vsts-extension.png)


## Steps ##

The first step - is to import the required `Clipboard` module from `Utils`.

```typescript
	import Clipboard = require("VSS/Utils/Clipboard");
``` 

Next step - Call `copyToClipboard(...)` method and pass the text you want to copy to clipboard.

```typescript
	let firstName: string = "John";
	let lastName: string = "Smith";
	Clipboard.copyToClipboard(`Full Name: ${firstName}, ${lastName}`);
```

## How to ensure browser supports clipboard copying ##

The `Clipboard` object provides two more methods (and some additional options) to check whether browser natively supports copying to clipboard.

- `supportsNativeCopy()` - boolean value indicating whether the current browser supports native clipboard access
- `supportsNativeHtmlCopy()` - boolean value indicating whether the current browser supports native clipboard access for HTML content

So for the safety check you might want to check the support before attempting to copy the text to clipboard like below.


```typescript
	let firstName: string = "John";
	let lastName: string = "Smith";

	if (Clipboard.supportsNativeCopy) {
            Clipboard.copyToClipboard(`Full Name: ${firstName}${lastName}`);
    }
```

## How to copy text as HTML ##

`copyToClipboard(...)` method takes additional parameter of type `IClipboardOptions`. This `IClipboardOptions` object has additional properties and one of those is `copyAsHtml`. Setting this to true allows text to be copied as HTML.


```typescript
Clipboard.copyToClipboard("<b>My Text</b>", <Clipboard.IClipboardOptions>{
            copyAsHtml: true
        });
```

**P.S : I could not get this working for some reason. The copied text always came as in plain text for me - which I will need to investigate further.**

That's it for this post. Thanks for reading.

