---
layout: post          #important: don't change this
title: "VSO Status Inspector - A walk through on creating a Visual Studio extension to track VSO Status"
date: 2015-05-31 10:36:00
author: Utkarsh Shigihalli
tags: [vsts, "vsts extensions"]
categories:
- blog                #important: leave this here
- "visual studio extensibility" 
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---
 
Recently I wrote a Visual Studio extension called [VSO Status Inspector](https://visualstudiogallery.msdn.microsoft.com/e87c82b9-dced-4fe2-9a40-f90139c56882) to monitor the status of [Visual Studio Online (VSO)](https://www.visualstudio.com/en-us/products/what-is-visual-studio-online-vs). If you haven’t yet I would encourage you to [**download**](https://visualstudiogallery.msdn.microsoft.com/e87c82b9-dced-4fe2-9a40-f90139c56882) it from the Visual Studio extension gallery. It supports both Visual Studio 2013 and 2015. 

<!--more-->


> **This post originally appeared on [Microsoft ALM Rangers Blog](http://blogs.msdn.com/b/visualstudioalmrangers/archive/2015/05/20/vso-status-inspector-a-walk-through-on-creating-a-visual-studio-extension-to-track-vso-status-by-utkarsh-shigihalli.aspx)**.


VSO Status Inspector extension, polls Visual Studio [Support Overview](https://www.visualstudio.com/en-us/support/support-overview-vs.aspx) page periodically and parses the overall status into an icon that is rendered on the Visual Studio status bar. 

<b> The extension targets to raise awareness of VSO issues to developers working in the IDE. While what the extension does is trivial, there is a lot happening under the hood to hook into the IDE. In this blog post I endeavour to walk you through the mechanics of developing this extension and integrating it with different parts of Visual Studio like status bar, output window and options window.
</b>

![Alt text](/images/screenshots/utkarsh/vso_status_inspector.png "VSO status Inspector")

> I assume that you are already aware of basics of writing Visual Studio extensions and know how to create a Visual Studio Package. If you are new to extending Visual Studio, suggest you to start from this [**page**](https://msdn.microsoft.com/en-us/library/dd885119.aspx) on MSDN.

For developing this extension I have used following components,

- Visual Studio 2013 Ultimate (you can also use [Visual Studio Community edition](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx) which is **free**)
- [Visual Studio SDK 2013](https://www.microsoft.com/en-gb/download/details.aspx?id=40758)
- [HTML Agility Pack](https://www.nuget.org/packages/HtmlAgilityPack)

I will break this blog post in to following sections,

1. How to integrate with Visual Studio status bar and display a custom icon
2. How to write messages to VS Output window
3. How to integrate with Options window
4. Parsing the VSO status from support view page
5. Putting everything together

## 1. How to integrate with Visual Studio status bar and display a custom icon
Visual Studio status bar is one of the important components of Visual Studio IDE. It provides subtle but clear visual indications on the current context and state of the Visual Studio IDE. For example, status bar turns orange when you are debugging, turns blue when you are loading a solution and violet color when its idle. 

![Alt text](/images/screenshots/utkarsh/vso_status_inspector_statusbar.png "Statusbar")


Internally, Visual Studio status bar is composed of four different regions as in the screen shot below.
![Alt text](/images/screenshots/utkarsh/vso_status_inspector_statusbar_regions.png "Statusbar regions")

`VSO Status Inspector` extension uses Animated icon area as we display an icon based on the VSO status. So let's see how to do that.

In our extension package class, we first need to get the instance of status bar. Visual Studio SDK (VS SDK) exposes interface [IVsStatusbar](http://msdn.microsoft.com/en-us/library/Microsoft.VisualStudio.Shell.Interop.IVsStatusbar%28v=vs.110%29.aspx "IVsStatusbar") to access the VS status bar. This interface provides different methods for accessing the different status bar regions. So to get the instance of status bar we use `GetService` method of our extension's Package class and get instance of `IVsStatusBar`.

I am declaring a property which will return the instance of `IVsStatusBar`.

```csharp
public IVsStatusbar StatusBar
{
	get
	{
    	if (bar == null)
    	{
        	bar = GetService(typeof(SVsStatusbar)) as IVsStatusbar;
    	}
    	return bar;
	}
}
```

Once we have the status bar instance, we can start calling the methods exposed by interface to perform actions on the status bar.

For this extension I have used `Animation` method as we need to display a icon in animation area. **Note** that VSSDK provides few default icons (like Build, Save etc) which can also be used with `Animation` method.

```csharp
object icon = (short)Microsoft.VisualStudio.Shell.Interop.Constants.SBAI_Deploy;
StatusBar.Animation(1, ref icon);
```

To display a custom icon, we need to take the custom icon we want to display and create a GDI bitmap and pass the reference to the `Animation` method - which is exactly what we are doing in below code snippet.

```csharp
IntPtr _hdcBitmap = IntPtr.Zero;
Bitmap b = ResizeImage(icon, 16);
_hdcBitmap = b.GetHbitmap();
object hdcObject = (object)_hdcBitmap;
StatusBar.Animation(1, ref hdcObject);
```


`GetHbitmap()` method converts the image to GDI bitmap object from the given `System.Drawing.Bitmap` object. The `ResizeImage(...)` method alters the given image to the size defined (16px in this case) and draws the high quality image and returns the bitmap. 
 
```csharp
public static Bitmap ResizeImage(Bitmap imgToResize, int newHeight)
{
    int sourceWidth = imgToResize.Width;
    int sourceHeight = imgToResize.Height;
    float nPercentH = ((float)newHeight / (float)sourceHeight);
    int destWidth = Math.Max((int)Math.Round(sourceWidth * nPercentH), 1); 
    int destHeight = newHeight;
    Bitmap bitmap = new Bitmap(destWidth, destHeight);
    using (Graphics graphics = Graphics.FromImage(bitmap))
    {
        graphics.SmoothingMode = SmoothingMode.HighQuality;
        graphics.InterpolationMode = InterpolationMode.HighQualityBicubic;
        graphics.PixelOffsetMode = PixelOffsetMode.HighQuality;
        graphics.DrawImage(imgToResize, 0, 0, destWidth, destHeight);
    }
    return bitmap;
}
```

## 2. How to write messages to Output window

`VSO Status Inspector` extension also outputs the complete information retrieved from the support overview page to the output window. 

![Alt text](/images/screenshots/utkarsh/vso_status_inspector_output.png "Output window")

Output window consists of different panes, which can be selected from the drop down. If you notice screen shot above, we have selected custom pane called `VSO Status Inspector`. A custom pane is used to display information only related to this extension.

To interact with output window, VS SDK provides another interface [IVsOutputWindow](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.interop.ivsoutputwindow.aspx "IVsOutputWindow"). Lets access that by defining another property as below

```csharp
public IVsOutputWindow OutputWindow
{
    get
    {
        if (_outputWindow == null)
        {
            _outputWindow = (IVsOutputWindow)GetService(typeof(SVsOutputWindow));
            return _outputWindow;
        }
        return _outputWindow;
    }
}
```

Now to write to the output window we use the below code snippet.

```csharp
private Guid _paneGuid = new Guid("{170638A1-CFD7-47C8-975A-FBAA9E532AD5}");
private void WriteToOutputWindow(string message)
{
    IVsOutputWindowPane outputPane;
    OutputWindow.GetPane(ref _paneGuid, out outputPane);

    if (outputPane == null)
    {
        // Create a new pane if not found
        OutputWindow.CreatePane(ref _paneGuid, EXTENSION_NAME, 
				Convert.ToInt32(true), Convert.ToInt32(false));
    }

    // Retrieve the new pane.
    OutputWindow.GetPane(ref _paneGuid, out outputPane);

    outputPane.OutputStringThreadSafe(string.Format("[{0}]\t{1}", 
				DateTime.Now.ToString("hh:mm:ss tt"), message));
    outputPane.OutputStringThreadSafe(Environment.NewLine);
}
```

In the above code we first try to get our custom (VSO Status Inspector) pane defined by a unique guid. If we cannot find any pane with our guid, we create a new pane. Finally we write the message passed via the parameter to the output window.

## 3. How to integrate with Options window
By default, `VSO Status Inspector` polls for status every 60 seconds. However, we allow user to customize this interval in our extension. To do that user goes to `Tools -> Options -> VSO Inspector` and can change the interval.

![Alt text](/images/screenshots/utkarsh/vso_status_inspector_options.png "Options window")

To achieve this, we need to integrate our extension with Visual Studio options window. To do that, first we need to define a custom class and inherit from [DialogPage](http://msdn.microsoft.com/en-IN/library/microsoft.visualstudio.shell.dialogpage.aspx) class of VS SDK.

```csharp
[ClassInterface(ClassInterfaceType.AutoDual)]
[CLSCompliant(false), ComVisible(true)]
public class VSOStatusInspectorOptions : DialogPage
{
    private int _interval = 60;
    [Category("General")]
    [DisplayName(@"Polling Interval (in seconds)")]
    [Description("Number of seconds between each poll.")]
    public int Interval
    {
        get { return _interval; }
        set { _interval = value; }
    }
}
```

The class above is simple with one property to hold the interval. If no value is specified we initialize interval as 60 seconds. Note that, the property has few annotations like category (under which this property will be visible), display name (display name for this property) and description (displayed in the help section under options window).

Finally, we also need to let Visual Studio know that our extension provides a options window, so that Visual Studio makes certain registry changes when our extension is installed. We do that, by decorating our package class with attribute [ProvideOptionPage](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.provideoptionpageattribute.aspx) as below

```csharp
[ProvideOptionPage(typeof(VSOStatusInspectorOptions), EXTENSION_NAME, "General", 0, 0, true)]
public sealed class VSOStatusInspectorPackage : Package
{
	private const string EXTENSION_NAME = "VSO Status Inspector";
	...
}
```
Finally, the unterval value set in the option window can be accessed in the extension as below.

```csharp
//get interval from options
var _options = (VSOStatusInspectorOptions)GetDialogPage(typeof(VSOStatusInspectorOptions));
var interval = _options.Interval;
```

## 4. Parsing the VSO status from support view page
As noted during the beginning of this article, the status is retrieved from [VSO Support Overview](https://www.visualstudio.com/en-us/support/support-overview-vs.aspx) page. The support page contains a `div` with status information and an icon. The markup is as below.
 
```html
<div class="TfsServiceStatus">
	<div data-fragmentname="StatusAvailable" id="Fragment_StatusAvailable" xmlns="...">
	  <div class="DetailedImage" style="position: relative">
	    <img id="GREEN" alt="Green-Service is up" src=".../IC711323.png" 
			title="Green-Service is up" xmlns="">
	    <div class="RichText" style="position:absolute;top:18px;left:62px" xmlns="...">
	      <h1 xmlns="">Visual Studio Online is up and running</h1>
	      <p xmlns="">Everything is looking good</p>
	      <p xmlns="">For details and history, check out the 
			<a href="http://blogs.msdn.com/b/vsoservice/">
			Visual Studio Service Blog.</a></p>
	    </div>
	  </div>
	</div>
</div>
```

To parse the above markup, I used very popular HTML parser - [HtmlAgilityPack](https://www.nuget.org/packages/HtmlAgilityPack).

The HtmlAgilityPack library makes parsing HTML very easy. We parse the above HTML from support overview page is as below.

```csharp
HtmlWeb htmlWeb = new HtmlWeb();
HtmlDocument doc = htmlWeb.Load("..."); //load support url - text trimmed for formatting

var div = doc.DocumentNode.SelectSingleNode("//div[@class='TfsServiceStatus']");
var img = div.SelectSingleNode("//img[@id]");
var h1 = div.SelectSingleNode("//div[@class='RichText']/h1");
var p = div.SelectSingleNode("//div[@class='RichText']/p");
```
In the code above, we are loading the page markup in to a document and then getting the `id` and other tags under `div` with `class` named `TfsServiceStatus`. I decide the status of VSO based on the image's `id` attribute used in the `img` tag and then also get the text within header (`h1`) and paragraph (`p`) tags to display it in VS output window.

So code for displaying different icon is as below. That is if `img` contains `GREEN` icon, that means VSO is up and so on.

```csharp
if (imageId == "GREEN")
	// display green icon
else if (imageId == "YELLOW")
 	// display yellow icon
else if (imageId == "RED")
	// display red icon
else
	// display different icon when status cannot be identified
```

## 5. Putting everything together

Now, only two more steps remain in our extension 
- Polling the status page to get up-to-date status and, 
- Auto load our extension when Visual Studio is launched.

### Polling the status page to get up to date status
To poll for status, we first get the interval defined in the Options page and then initialize the timer in package's `Initialize` method as below.

```csharp
protected override void Initialize()
{
    base.Initialize();
    //Set to unknown icon till we find the status
    SetIcon(Resources.unknown);
    //get interval from options
    _options = (VSOStatusInspectorOptions)
            GetDialogPage(typeof(VSOStatusInspectorOptions));
    //call the timer code first without waiting for timer trigger
    OnTimerTick(null, null);
    //Set the timer
    var timer = new Timer();
    timer.Interval = TimeSpan.FromSeconds(_options.Interval).TotalMilliseconds;
    timer.Elapsed += OnTimerTick;
    timer.Start();
}
```

### Auto load our extension when Visual Studio is launched
Finally, Visual Studio by default loads the extensions on demand or when context is initialized for which extension is dependent on. For example, when you click a menu item in your extension, your extension is loaded *on demand* as menu click requires your extension to be initialized.  Similarly, if you perform an action triggering a context change in which your extension operates (like you trigger a solution load) and all extensions depending on `SolutionExists` context are loaded. 

For our extension, we wanted to load the extension always. That is either when Visual Studio is opened in empty environment (no solution) or when user is working in a solution (solution exists). So we decorate our package class with two more attributes to set these context as below.

```csharp
[ProvideAutoLoad(UIContextGuids80.NoSolution)]
[ProvideAutoLoad(UIContextGuids80.SolutionExists)]
```

Our extension is now loaded by Visual Studio always.

That's it for this post. You can download the complete source code from [**GitHub**](https://github.com/onlyutkarsh/VSOStatusInspector/) 

I hope you got a good overview of how every part (Status bar, Options window and output window for example) of Visual Studio can be extended seamlessly using Visual Studio SDK. So until next time, happy extending Visual Studio.

## Further reading ##

- [Using Visual Studio status bar in your extensions](http://geekswithblogs.net/onlyutkarsh/archive/2013/08/11/using-visual-studio-status-bar-in-your-extensions.aspx)

- [Integration with Visual Studio Options Window using custom controls](http://geekswithblogs.net/onlyutkarsh/archive/2013/06/30/integration-of-options-window-in-visual-studio-extension-with-custom.aspx)
