---
title: Win10 UWP初始屏幕设计
tags:
  - UWP
categories:
  - - 开发笔记
date: 2017-09-02 02:30:30
---

首先说一下初始屏幕设计的必要性。

*   如果UWP应用的启动后需要加载的资源比较多，初始屏幕过后，有较长的等待时间，这时一个延长的初始屏幕可以很好的解决这个问题。
*   系统默认的初始屏幕过于单调简单，一个更好的初始屏幕，会使app更加优秀。
<!-- more -->
初始屏幕的设置，在widows开发者文档中有详细介绍（[延长显示初始屏幕的时间](https://docs.microsoft.com/zh-cn/windows/uwp/launch-resume/create-a-customized-splash-screen)），但是按照这种方法做出来的有一个问题就是手机界面兼容性不好，会出现错位等问题，在博客园MS-UAP的“[人群中看到你的第一眼-初始屏幕](http://www.cnblogs.com/ms-uap/p/5314026.html)”中有解决办法。 完成后的代码如下：
```
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices.WindowsRuntime;
using Windows.Foundation;
using Windows.Foundation.Collections;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Controls.Primitives;
using Windows.UI.Xaml.Data;
using Windows.UI.Xaml.Input;
using Windows.UI.Xaml.Media;
using Windows.UI.Xaml.Navigation;
using Windows.ApplicationModel.Activation;
using Windows.UI.Core;
using Windows.UI.ViewManagement;
using PasswordTest.Helpers;
using Windows.UI;
using System.Reflection;
// https://go.microsoft.com/fwlink/?LinkId=234238 上介绍了“空白页”项模板

namespace PasswordTest
{
///

/// 可用于自身或导航至 Frame 内部的空白页。
///

public sealed partial class SplashPage : Page
{
internal Rect splashImageRect; // 用来获取屏幕显示区域的大小
private SplashScreen splash; // Variable to hold the splash screen object.
internal bool dismissed = false; //追踪启动画面解除状态的变量。
internal Frame rootFrame;

// Define methods and constructor
public SplashPage(SplashScreen splashscreen, bool loadState)
{
InitializeComponent();

// Listen for window resize events to reposition the extended splash screen image accordingly.
// This ensures that the extended splash screen formats properly in response to window resizing.
Window.Current.SizeChanged += new WindowSizeChangedEventHandler(ExtendedSplash\_OnResize);

splash = splashscreen;
if (splash != null)
{
// 注册一个事件处理程序，以便在启动屏幕被关闭时执行。
splash.Dismissed += new TypedEventHandler<SplashScreen, Object>(DismissedEventHandler);

// 检索启动屏幕图像的窗口坐标。
splashImageRect = splash.ImageLocation;
PositionImage();

// 如果使用，请包括一个定位进度控制的方法。
PositionRing();
}

// Create a Frame to act as the navigation context
rootFrame = new Frame();
}

void PositionImage()
{
// desktop
if (Windows.System.Profile.AnalyticsInfo.VersionInfo.DeviceFamily.Equals("windows.desktop", StringComparison.CurrentCultureIgnoreCase))
{
extendedSplashImage.SetValue(Canvas.LeftProperty, splashImageRect.X);
extendedSplashImage.SetValue(Canvas.TopProperty, splashImageRect.Y);
extendedSplashImage.Height = splashImageRect.Height;
extendedSplashImage.Width = splashImageRect.Width;
}
// mobile
else if (Windows.System.Profile.AnalyticsInfo.VersionInfo.DeviceFamily.Equals("windows.mobile", StringComparison.CurrentCultureIgnoreCase))
{
if (Windows.Foundation.Metadata.ApiInformation.IsTypePresent("Windows.UI.ViewManagement.StatusBar"))
{
// 需要引用Windows Mobile Extensions for the UWP
StatusBar sb = StatusBar.GetForCurrentView();
// 调整为透明，也可以用启动屏幕的背景色+不透明
sb.BackgroundOpacity = 0;
sb.ForegroundColor = Colors.Black;

// 撑满手机屏幕
Windows.UI.ViewManagement.ApplicationView.GetForCurrentView().SetDesiredBoundsMode(Windows.UI.ViewManagement.ApplicationViewBoundsMode.UseCoreWindow);
}
// 获取一个值，该值表示每个视图（布局）像素的原始（物理）像素数。
double density = Windows.Graphics.Display.DisplayInformation.GetForCurrentView().RawPixelsPerViewPixel;

extendedSplashImage.SetValue(Canvas.LeftProperty, splashImageRect.X / density);
extendedSplashImage.SetValue(Canvas.TopProperty, splashImageRect.Y / density);
extendedSplashImage.Height = splashImageRect.Height / density;
extendedSplashImage.Width = splashImageRect.Width / density;
}
}

void PositionRing()
{
splashProgressRing.SetValue(Canvas.LeftProperty, splashImageRect.X + (splashImageRect.Width \* 0.5) - (splashProgressRing.Width \* 0.5));
splashProgressRing.SetValue(Canvas.TopProperty, (splashImageRect.Y + splashImageRect.Height + splashImageRect.Height \* 0.1));
}
// 包括当系统从启动屏幕转换到扩展启动屏幕（应用程序的第一个视图）时要执行的代码。
async void DismissedEventHandler(SplashScreen sender, object e)
{
dismissed = true;
// 在这儿完成应用设置的操作

//在应用设置完成后，自动跳转到主页
await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
{
//操作UI的代码
DismissExtendedSplash();
});
}

void DismissExtendedSplash()
{
// 导航到mainpage
rootFrame.Navigate(typeof(MainPage));
// 将frame放在当前窗口中
Window.Current.Content = rootFrame;
}

void ExtendedSplash\_OnResize(Object sender, WindowSizeChangedEventArgs e)
{
// 安全更新extended splash screen image的坐标.
//当用户调整窗口大小时，将执行此功能。
if (splash != null)
{
// Update the coordinates of the splash screen image.
splashImageRect = splash.ImageLocation;
PositionImage();

// If applicable, include a method for positioning a progress control.
// PositionRing();
}
}

//测试按钮
private void DismissSplashButton\_Click(object sender, RoutedEventArgs e)
{
DismissExtendedSplash();
}

//async void RestoreStateAsync(bool loadState)
//{
// if (loadState)
// {
// // 代码可以在这里加载你的应用程序的状态

// }
//}
}

}

但是，我按照这两篇帖子做后发现在初始界面加载完后需要导航到主页时，却发生异常。 提示跨线程操作UI，经过一番研究后，解决方法如下：

// 包括当系统从启动屏幕转换到扩展启动屏幕（应用程序的第一个视图）时要执行的代码。
async void DismissedEventHandler(SplashScreen sender, object e)
{
dismissed = true;
// 在这儿完成应用设置的操作

//在应用设置完成后，自动跳转到主页
await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
{
//操作UI的代码
DismissExtendedSplash();
});
}
```
非UI线程访问UI参考这篇文章。[点击链接](https://lindexi.oschina.io/lindexi/post/win10-uwp-%E9%9D%9Eui%E7%BA%BF%E7%A8%8B%E8%AE%BF%E9%97%AE-ui.html) 这样一个延长的初始屏幕就完成了，并且适配手机和pc，其他的就看自己发挥了。 还有一个问题是，在手机上ProgressRing不显示的问题，目前还没时间研究，等有结果了，再更新~ 更新：手机上不显示ProgressRing问题的原因找到了，是因为ProgressRing的位置有问题，就是它的坐标超出屏幕的显示范围了，所以看不到，具体解决方法及位置计算方法，详见[UWP手机启动屏幕控件位置计算](https://www.yzj0308.com/uwp%E6%89%8B%E6%9C%BA%E5%90%AF%E5%8A%A8%E5%B1%8F%E5%B9%95%E6%8E%A7%E4%BB%B6%E4%BD%8D%E7%BD%AE%E8%AE%A1%E7%AE%97/)。