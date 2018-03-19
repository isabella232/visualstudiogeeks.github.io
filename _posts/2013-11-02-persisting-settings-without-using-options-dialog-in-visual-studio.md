---
layout: post          #important: don't change this
title: "Persisting settings without using Options dialog in Visual Studio"
date: 2013-11-02 17:14:47
author: utkarsh
tags: [Extensions, VisualStudio]
categories:
- "VisualStudio"
- "Shell"
- "Extensions"
description: "Persisting settings without using Options dialog in Visual Studio"
image:        #place image (850x450)
thumb: thumb-icon-utkarsh.jpg    #place thumbnail (70x70)
---
In one of my previous [blog post](http://geekswithblogs.net/onlyutkarsh/archive/2013/06/30/integration-of-options-window-in-visual-studio-extension-with-custom.aspx) we have seen persisting settings using Visual Studio's options dialog. Visual Studio options has many advantages in automatically persisting user options for you. 

However, during our latest [Team Rooms extension](http://visualstudiogallery.msdn.microsoft.com/c1bf5e4f-5436-465d-87da-09b2f15ff061) development, we decided to provide our users; ability to use our preferences directly from Team Explorer. The main reason was that we had only one simple option for user and we thought it is cumbersome for user to go to Tools –> Options dialog to change this. Another reason was, we wanted to highlight this setting to user as soon as he is using our extension.

![image]({{site.url}}/images/screenshots/utkarsh//2013_11_02_persisting_settings_without_using_Image1.png "image")

So if you are in such a scenario where you do not want to use VS options window, but still would like to persist the settings, this post will guide you through.

Visual Studio SDK provides two ways to persist settings in your extensions. One is using DialogPage as shown in my previous post. Another way is to use by implementing [IProfileManager](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.iprofilemanager.ASPX) interface which I will explain in this post. Please note that the class implementing IProfileManager should be independent class. This is because, VS instantiates this class during Tools –> Import and Export Settings.

IProfileManager provides 2 different sets of methods (total 4 methods) to persist the settings. They are

1.  `LoadSettingsFromXml` and `SaveSettingsToXml` – Implement these methods to persist settings to disk from VS settings storage. The VS will persist your settings along with other options to disk. 
2.  `LoadSettingsFromStorage` and `SaveSettingsToStorage` – Implement these methods to persist settings to local storage, usually it be registry. VS calls `LoadSettingsFromStorage` method when it is initializing the package too.   

We are going to use the 2nd set of methods for this example. First, we are creating a separate class file called UserOptions.cs. Please note that, we also need to implement [IComponent](http://msdn.microsoft.com/en-us/library/system.componentmodel.icomponent.ASPX), which can be done by inheriting [Component](http://msdn.microsoft.com/en-us/library/system.componentmodel.component.ASPX) along with `IProfileManager`. 

```cs
[ComVisible(true)]
[Guid("XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX")]
public class UserOptions : Component, IProfileManager
{
   private const string SUBKEY_NAME = "TForVS2013";
   private const string TRAY_NOTIFICATIONS_STRING = "TrayNotifications";
   ...
}
```

Define the property so that it can be used to set and get from other classes.

```cs
public bool TrayNotifications { get; set; }
```

Implement the members of `IProfileManager`.

```cs
public void LoadSettingsFromStorage()
{
      RegistryKey reg = null;
      try
      {
           using (reg = Package.UserRegistryRoot.OpenSubKey(SUBKEY_NAME))
           {
                if (reg != null)
                {
                     // Key already exists, so just update this setting.
                     TrayNotifications = Convert.ToBoolean(reg.GetValue(TRAY_NOTIFICATIONS_STRING, true));
                }
           }
      }
      catch (TeamRoomException exception)
      {
           TrayNotifications = true;
           ExceptionReporting.Report(exception);
      }
      finally
      {
           if (reg != null)
           {
               reg.Close();
           }
      }
}

public void LoadSettingsFromXml(IVsSettingsReader reader)
{
      reader.ReadSettingBoolean(TRAY_NOTIFICATIONS_STRING, out _isTrayNotificationsEnabled);
      TrayNotifications = (_isTrayNotificationsEnabled == 1);
}

public void ResetSettings()
{
}

public void SaveSettingsToStorage()
{
      RegistryKey reg = null;
      try
      {
           using (reg = Package.UserRegistryRoot.OpenSubKey(SUBKEY_NAME, true))
           {
                if (reg != null)
                {
                    // Key already exists, so just update this setting.
                    reg.SetValue(TRAY_NOTIFICATIONS_STRING, TrayNotifications);
                }
                else
                {
                    reg = Package.UserRegistryRoot.CreateSubKey(SUBKEY_NAME);
                    reg.SetValue(TRAY_NOTIFICATIONS_STRING, TrayNotifications);
                }
           }
      }
      catch (TeamRoomException exception)
      {
           ExceptionReporting.Report(exception);
      }
      finally
      {
           if (reg != null)
           {
                reg.Close();
           }
      }
}

public void SaveSettingsToXml(IVsSettingsWriter writer)
{
      writer.WriteSettingBoolean(TRAY_NOTIFICATIONS_STRING, TrayNotifications ? 1 : 0);
}
```
Let me elaborate on the method implementation. The Package class provides UserRegistryRoot (which is `HKCU\Microsoft\VisualStudio\12.0` for VS2013) property which can be used to create and read the registry keys. So basically, in the methods above, I am checking if the registry key exists already and if not, I simply create it. Also, in case there is an exception I return the default values. If the key already exists, I update the value. Also, note that you need to make sure that you close the key while exiting from the method. Very simple right?

Accessing and settings is simple too. We just need to use the exposed property.

```cs
UserOptions.TrayNotifications = true;
UserOptions.SaveSettingsToStorage();
```

Reading settings is as simple as reading a property.

```cs
UserOptions.LoadSettingsFromStorage();
var trayNotifications =  UserOptions.TrayNotifications;
```

Lastly, the most important step. We need to tell Visual Studio shell that our package exposes options using the UserOptions class. For this we need to decorate our package class with [ProvideProfile](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.shell.provideprofileattribute.aspx) attribute as below.

```cs
[ProvideProfile(typeof(UserOptions), "TForVS2013", "TeamRooms", 110, 110, false, DescriptionResourceID = 401)]
public sealed class TeamRooms : Microsoft.VisualStudio.Shell.Package
{
...
}
```

That's it. If everything is alright, once you run the package you will also see your options appearing in "Import Export settings" window, which allows you to export your options.

![image]({{site.url}}/images/screenshots/utkarsh//2013_11_02_persisting_settings_without_using_Image2.png "image")