<img src="https://raw.githubusercontent.com/Aldaviva/DarkNet/master/DarkNet/icon.png" height="23" alt="DarkNet logo" /> DarkNet
===

[![Nuget](https://img.shields.io/nuget/v/DarkNet?logo=nuget)](https://www.nuget.org/packages/DarkNet/) [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/Aldaviva/darknet/dotnetpackage.yml?branch=master&logo=github)](https://github.com/Aldaviva/DarkNet/actions/workflows/dotnetpackage.yml)

Enable native Windows dark mode for your WPF and Windows Forms title bars.

![WPF window with dark title bar](.github/images/demo-wpf.png)

<!-- MarkdownTOC autolink="true" bracket="round" autoanchor="true" levels="1,2,3,4" -->

- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
    - [Basics](#basics)
        - [Entry point](#entry-point)
        - [Methods](#methods)
        - [Themes](#themes)
    - [WPF](#wpf)
        - [On application startup](#on-application-startup)
        - [Before showing a new window](#before-showing-a-new-window)
        - [After showing a window](#after-showing-a-window)
        - [Complete example](#complete-example)
    - [Windows Forms](#windows-forms)
        - [On application startup](#on-application-startup-1)
        - [Before showing a new window](#before-showing-a-new-window-1)
        - [After showing a window](#after-showing-a-window-1)
        - [Complete example](#complete-example-1)
    - [Raw HWND](#raw-hwnd)
    - [Taskbar theme](#taskbar-theme)
- [Demos](#demos)
    - [WPF](#wpf-1)
    - [Windows Forms](#windows-forms-1)
- [Limitations](#limitations)
- [Acknowledgements](#acknowledgements)

<!-- /MarkdownTOC -->

<a id="requirements"></a>
## Requirements

- .NET runtime
    - .NET 5.0 or later
    - .NET Core 3.1 or later
    - .NET Framework 4.5.2 or later
- Windows
    - Windows 11 or later
    - Windows 10 version 1809 (October 2018 Update) or later
    - You can still run your program on earlier Windows versions as well, but the title bar won't turn dark
- Windows Presentation Foundation, Windows Forms, or access to the native window handle of other windows in your process

<a id="installation"></a>
## Installation

[DarkNet is available in NuGet Gallery.](https://www.nuget.org/packages/DarkNet/)

```ps1
dotnet add package DarkNet
```
```ps1
Install-Package DarkNet
```

<a id="usage"></a>
## Usage

<a id="basics"></a>
### Basics

<a id="entry-point"></a>
#### Entry point

The top-level interface of this library is **`Dark.Net.IDarkNet`**, which is implemented by the **`DarkNet`** class. A shared instance of this class is available from **`DarkNet.Instance`**, or you can construct a new instance with `new DarkNet()`.

<a id="methods"></a>
#### Methods

1. First, you may optionally call **`SetCurrentProcessTheme(Theme)`** to define a default theme for your windows, although it doesn't actually apply the theme to any windows on its own.

    This method also controls the theme of the application's context menu, which appears when you right-click the title bar of any of its windows. This means all windows in your app will have the same context menu theme, even if you set their window themes differently.
    
    If you don't call this method, any window on which you call `SetWindowTheme*(myWindow, Theme.Auto)` will inherit its theme from the operating system's default app theme, skipping this app-level default.
2. Next, you must call one of the **`SetWindowTheme*`** methods to actually apply a theme to each window. There are three methods to choose from, depending on what kind of window you have:
    |Window type|Method signature|
    |-|-|
    |WPF|`SetWindowThemeWpf(Window, Theme)`|
    |Windows Forms|`SetWindowThemeForms(Form, Theme)`|
    |Raw HWND|`SetWindowThemeRaw(IntPtr, Theme)`|

    If you don't call one of these methods on a given window, that window will always use the light theme, even if you called `SetCurrentProcessTheme` and set the OS default app mode to dark.

    *These are three separate methods instead of one overloaded method in order to prevent apps from having to depend on **both** WPF and Windows Forms when they only intend to use one, because an overloaded method signature can't be resolved when any of the parameters' types are missing.*

<a id="themes"></a>
#### Themes

This library uses the `Theme` enum to differentiate **`Dark`** mode from **`Light`** mode. You can set any window in your application to use whichever theme you want, they don't all have to be the same theme.

There is also an **`Auto`** theme value that allows the theme to be inherited from a higher level, falling back from the **window** to the **process** to the user account **default app theme**:

1. When a **window's** theme is `Dark` or `Light`, it uses that theme directly, set by`SetWindowTheme*`.
2. When a **window's** theme is `Auto`, it inherits from the **process's** theme set by `SetCurrentProcessTheme`.
3. When a **window and its process's** themes are both `Auto`, they inherit from the local user's operating system **default app theme** preferences (Settings › Personalization › Colors › Choose your default app mode).

##### Live updates

If your app is running with the `Auto` theme at both the window and process levels, and the user changes their OS default app mode in Settings, then your app will automatically render with the new theme, without you having to handle any events or restart the app.

Try the [demo apps](#demos) to see this behavior in action.

<a id="wpf"></a>
### WPF

<a id="on-application-startup"></a>
#### On application startup

Before showing **any** windows in your application, you may optionally call
```cs
IDarkNet.SetCurrentProcessTheme(theme);
```

A good place to call this is in an event handler for your Application's **`Startup`** event, or in its overridden **`OnStartup`** method.

By default, WPF apps have an `App.xaml.cs` class that inherits from `Application`. You can override `OnStartup` to initialize DarkNet:

```cs
// App.xaml.cs
using Dark.Net;

public partial class App: Application {

    protected override void OnStartup(StartupEventArgs e) {
        base.OnStartup(e);
        DarkNet.Instance.SetCurrentProcessTheme(Theme.Auto);
    }
}
```

<a id="before-showing-a-new-window"></a>
#### Before showing a new window

Before showing **each** window in your application, you have to set the theme for that window.

```cs
IDarkNet.SetWindowThemeWpf(window, theme);
```

A good place to call this is in the window's constructor **after the call to `InitializeComponent`**, or in an event handler for the window's **`SourceInitialized`** event.

If you call it too late (such as after the window is shown), the theme will not change.

```cs
// MainWindow.xaml.cs
using Dark.Net;

public partial class MainWindow {

    public MainWindow() {
        InitializeComponent();
        DarkNet.Instance.SetWindowThemeWpf(this, Theme.Auto);
    }

}
```

You must perform this step for **every** window you show in your application, not just the first one.

<a id="after-showing-a-window"></a>
#### After showing a window

After calling `SetWindowThemeWpf` for the first time and showing a new window, you may optionally make additional calls to `SetCurrentProcessTheme` or `SetWindowThemeWpf` multiple times to change the theme later. This is useful if you let users choose a theme in your app's settings.

Try the [demo apps](#demos) to see this behavior in action.

<a id="complete-example"></a>
#### Complete example

See the demo [`App.xaml.cs`](https://github.com/Aldaviva/DarkNet/blob/master/darknet-demo-wpf/App.xaml.cs) and [`MainWindow.xaml.cs`](https://github.com/Aldaviva/DarkNet/blob/master/darknet-demo-wpf/MainWindow.xaml.cs).

<a id="windows-forms"></a>
### Windows Forms

<a id="on-application-startup-1"></a>
#### On application startup

Before showing **any** windows in your application, you may optionally call
```cs
IDarkNet.SetCurrentProcessTheme(theme);
```

A good place to call this is at the **start of the `Main()` method** of your application.

```cs
// Program.cs
using Dark.Net;

internal static class Program {

    [STAThread]
    private static void Main() {
        DarkNet.Instance.SetCurrentProcessTheme(Theme.Auto);

        // Incomplete example; see below for complete example including showing your first window.
    }
}
```

<a id="before-showing-a-new-window-1"></a>
#### Before showing a new window

Before showing **each** window in your application, you have to set the theme for that window.

```cs
IDarkNet.SetWindowThemeForms(window, theme);
```

You must do this **before calling `Show()` or `Application.Run()`** to show the window. If you call it too late (such as after the window is shown), the theme will not change.

```cs
Form mainForm = new Form1();
DarkNet.Instance.SetWindowThemeForms(mainForm, Theme.Auto);
```

You must perform this step for **every** window you show in your application, not just the first one.

<a id="after-showing-a-window-1"></a>
#### After showing a window

After calling `SetWindowThemeForms` for the first time and showing a new window, you may optionally make additional calls to `SetCurrentProcessTheme` or `SetWindowThemeForms` multiple times to change the theme later. This is useful if you let users choose a theme in your app's settings.

Try the [demo apps](#demos) to see this behavior in action.

<a id="complete-example-1"></a>
#### Complete example

```cs
// Program.cs
using Dark.Net;

internal static class Program {

    [STAThread]
    private static void Main() {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        DarkNet.Instance.SetCurrentProcessTheme(Theme.Auto);

        Form mainForm = new Form1();
        DarkNet.Instance.SetWindowThemeForms(mainForm, Theme.Auto);

        Application.Run(mainForm);
    }

}
```

See also the demo [`Program.cs`](https://github.com/Aldaviva/DarkNet/blob/master/darknet-demo-winforms/Program.cs) and [`Form1.cs`](https://github.com/Aldaviva/DarkNet/blob/master/darknet-demo-winforms/Form1.cs).

<a id="raw-hwnd"></a>
### Raw HWND

If you want to change the theme of a window in your application that was not created with WPF or Windows Forms, you can also just pass the HWND of the window to `SetWindowThemeRaw(IntPtr, Theme)`.

<a id="taskbar-theme"></a>
### Taskbar theme

Windows introduced a preference to choose a dark or light taskbar in Windows 10 version 1903. This is controlled by Settings › Personalization › Colors › Choose your default Windows mode.

DarkNet exposes the value of this preference with the **`UserTaskbarThemeIsDark`** property, as well as the change event **`UserTaskbarThemeIsDarkChanged`**. You can use these to render a tray icon in the notification area that matches the taskbar's theme, and re-render it when the user preference changes.

<a id="demos"></a>
## Demos

You can download the following precompiled demos, or clone this repository and build the demo projects yourself using Visual Studio Community 2022.

<a id="wpf-1"></a>
### WPF

Download and run `darknet-demo-wpf.exe` from the [latest release](https://github.com/Aldaviva/DarkNet/releases).

Requires [.NET Desktop Runtime 6 x64](https://dotnet.microsoft.com/en-us/download/dotnet/6.0) or later.

![WPF window with dark title bar](.github/images/demo-wpf.png)

<a id="windows-forms-1"></a>
### Windows Forms

Download and run `darknet-demo-winforms.exe` from the [latest release](https://github.com/Aldaviva/DarkNet/releases).

Requires [.NET Framework 4.8](https://dotnet.microsoft.com/en-us/download/dotnet-framework) or later.

![Windows Forms window with dark title bar](.github/images/demo-winforms.png)

<a id="limitations"></a>
## Limitations
- This library only changes the theme of the title bar/window chrome/non-client area, as well as the system context menu (the menu that appears when you right click on the title bar, or left click on the title bar icon, or hit `Alt`+`Space`). It does not change the theme of the client area of your window. It is up to you to make that look different when dark mode is enabled. This is difficult with Windows Forms, [particularly on .NET Core](https://github.com/gitextensions/gitextensions/issues/9191).
- This library currently does not help you persist a user's choice for the mode they want your application to use across separate process executions. You can expose an option and persist that yourself, then pass the desired `Theme` value to the methods in this library.

<a id="acknowledgements"></a>
## Acknowledgements

- [Milan Burda](https://github.com/miniak) for explaining how to add this in Electron ([electron/electron #23479: [Windows] Title bar does not respect dark mode](https://github.com/electron/electron/issues/23479))
- [dyasta](https://stackoverflow.com/users/191514/dyasta) for [an explanation on Stack Overflow](https://stackoverflow.com/a/58547831/979493)
- [Richard Yu](https://github.com/ysc3839) for [implementing this in a C++ library](https://github.com/ysc3839/win32-darkmode)
- [Berrysoft](https://github.com/Berrysoft) for [implementing this in Mintty, the Cygwin Terminal emulator](https://github.com/mintty/mintty/issues/983) ([further discussion](https://github.com/mintty/mintty/pull/984))