---
layout: post          #important: don't change this
title: "Options for displaying modal dialogs in Visual Studio extensions"
date: 2016-01-27 23:45:00 
author: utkarsh
tags: ["Extensions", "VisualStudio"]
categories:
- "extensibility"
- "visual studio"
 
img:        #place image (850x450) with this name in /images/screenshots
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70) with this name in /images/screenshotsthumbs/
---
In this blog post, we will explore different options for displaying WPF modal dialog in Visual Studio extensions.
<!--more-->

Visual Studio SDK allows making a modal dialog from any XAML window with some small code change as below. The code below gets the handle of the owner window via UI shell service and displays the dialog as a modal window.

```csharp
IVsUIShell uiShell = (IVsUIShell)ServiceProvider.GetService(typeof(SVsUIShell));
MyDialog myDialog = new MyDialog();
//get the owner of this dialog
IntPtr hwnd;
uiShell.GetDialogOwnerHwnd(out hwnd);
myDialog.WindowStartupLocation = System.Windows.WindowStartupLocation.CenterOwner;
uiShell.EnableModeless(0);
try
{
    WindowHelper.ShowModal(myDialog, hwnd);
}
finally
{
    // This will take place after the window is closed.
    uiShell.EnableModeless(1);
}
```

Now, as a dialog window, you may want to hide maximize and minimize buttons for the dialog. There are two options to hide maximize/minimize buttons.

 - By changing the window style
 - By using the VS SDK.

## Change the window style ##
This is one of the simplest options and also used in normal WPF programs. In the XAML file, you can set the `WindowStyle` as `ToolWindow` and you will get a dialog (with only close button) as below.

![Alt text](/images/screenshots/utkarsh/xaml_dialog_toolwindow.png)

#### Pros ####
- Easy and quick


## Use VS SDK ##
VS SDK provides a native XAML dialog window. This option ensures the UI and functionality of the dialog are compatible with Visual Studio. You need to make few changes to the WPF window (XAML) file as below.

- Based on your Visual Studio version you are targeting import `Microsoft.VisualStudio.Shell.{version}.0.dll` file. For example, an extension targeting VS 2015, I imported `Microsoft.VisualStudio.Shell.14.0.dll`
-  Add below namespace to XAML 

```xsd
xmlns:platformUi="clr-namespace:Microsoft.VisualStudio.PlatformUI;assembly=Microsoft.VisualStudio.Shell.14.0"
```
-  Modify the markup to change the `Window` to `DialogWindow`
  ![Alt text](/images/screenshots/utkarsh/xaml_dialog_window_diff.png)
- Instead of inheriting code file (*.xaml.cs) from Window class, inherit from `DialogWindow`class of PlatformUI.
- Finally, in the command from where you want to trigger this dialog, call the dialog as `ShowModal()` method.

*See above changes on* [**GitHub**](https://github.com/onlyutkarsh/XamlDialogInVSExtensionDemo/commit/616a945e3399e4869c6cd4ef28cb5b377495559b)



This will show the dialog similar to what we have seen above screenshot. However, you can make use of additional properties available to control the window properties. Specifically, you can

```csharp
var xamlDialog = new ();
xamlDialog.HasMinimizeButton = false;
xamlDialog.HasMaximizeButton = true;
xamlDialog.ShowModal();
```
...and you will see the dialog as below. As you can see you have maximize and close buttons, but minimize is disabled. 

![Alt text](/images/screenshots/utkarsh/xaml_dialog_platformui.png)

### Display Help button on the dialog ###

Another **great** feature of this dialog based on SDK is that it provides support for the help button. That means you can create this dialog and also display a help button to show help. Enabling that is easy too.

![Alt text](/images/screenshots/utkarsh/xaml_dialog_platformui_helpbutton.png)

- The DialogWindow class provides another parameterized constructor as below, add that to your *.xaml.cs file.
 
```csharp
public XamlDialog(string helpTopic) : base(helpTopic)
{
    InitializeComponent();
}
```
- Call the dialog with the parameterized constructor as below by passing the appropriate help topic as below.

```csharp
var xamlDialog = new XamlDialog ("Microsoft.VisualStudio.PlatformUI.DialogWindow");
xamlDialog.HasMinimizeButton = false;
xamlDialog.HasMaximizeButton = false;
xamlDialog.ShowModal();
```


**Note: The help button is only shown when both maximize and minimize buttons are *hidden***

#### Pros ####
- Functionality and UI are native to Visual Studio.
- Ability to integrate with help
- Control over the maximize and minimize buttons

A sample code demoing this functionality is on [**GitHub**](https://github.com/onlyutkarsh/XamlDialogInVSExtensionDemo). Running it adds `Open XAML Dialog` under Visual Studio `Tools` menu in experimental instance of Visual Studio - Clicking the menu, displays the dialog with a help button. 