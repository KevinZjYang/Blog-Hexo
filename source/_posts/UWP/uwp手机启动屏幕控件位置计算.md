---
title: Win10 UWP 手机启动屏幕控件位置计算
tags:
  - UWP
  - Windows 10
categories:
  - - 开发笔记
date: 2017-09-08 02:15:03
---

在[uwp初始屏幕设计](https://www.yzj0308.com/uwp%E5%88%9D%E5%A7%8B%E5%B1%8F%E5%B9%95%E8%AE%BE%E8%AE%A1/)中，最后在手机上调试，发现ProgressRing及文字都不显示的问题。最后，经过调试发现是它们的位置超出屏幕的显示范围了，所以才会看不到。下面我将已实际的例子，说明我自己的手机上计算位置方法。 以我的手机Lumia640xl为例，手机的分辨率是1280x720。
<!-- more -->
所以使用`// 检索启动屏幕图像的窗口坐标。 splashImageRect = splash.ImageLocation;` 获取的值：X=0,Y=0,Width = 720,Height = 1280。经过测试，坐标系的原点在屏幕左上角，以下面的方法计算，
```
void PositionRing()
{
    splashProgressRing.SetValue(Canvas.LeftProperty, splashImageRect.X + (splashImageRect.Width \* 0.5) - (splashProgressRing.Width \* 0.5));
    splashProgressRing.SetValue(Canvas.TopProperty, (splashImageRect.Y + splashImageRect.Height + splashImageRect.Height \* 0.1));
}
```
得出的结果是splashProgressRing的x= 336(splashProgressRing.Width = 48），splashProgressRing的Y= 1480；而实际手机屏幕的有效像素是多少呢？
```
double density = Windows.Graphics.Display.DisplayInformation.GetForCurrentView().RawPixelsPerViewPixel;
```
使用这个方法可以获取，手机屏幕有效像素和分辨率的一个转换值（我个人的理解），调试得到的这个值为1.75（不同手机这个值可能会不一样，没试过，好像是和显示缩放有关系）。计算一下，得到有效像素约等于411x731,所以在屏幕上肯定看不到（336,1428）。知道这个换算方法以后，就很好解决了，只需要把控件按照现有的坐标系计算坐标就可以了（x最大值411，y最大值731）。但是为了适配不同分辨率的手机，因此，还是建议尽量不要直接给出具体的坐标，而是有一定的参照。我使用下面的算式，使得ProgressRing的位置正常了。（但是由于手里只有这一种型号的手机，所以，不肯定是否完美解决了问题）
```
void PositionRing()
{
//desktop
if (Windows.System.Profile.AnalyticsInfo.VersionInfo.DeviceFamily.Equals("windows.desktop", StringComparison.CurrentCultureIgnoreCase))
{
splashProgressRing.SetValue(Canvas.LeftProperty, splashImageRect.X + (splashImageRect.Width \* 0.5) - (splashProgressRing.Width \* 0.5));
splashProgressRing.SetValue(Canvas.TopProperty, (splashImageRect.Y + splashImageRect.Height + splashImageRect.Height \* 0.1));

}
//mobile
else if (Windows.System.Profile.AnalyticsInfo.VersionInfo.DeviceFamily.Equals("windows.mobile", StringComparison.CurrentCultureIgnoreCase))
{
    double density = Windows.Graphics.Display.DisplayInformation.GetForCurrentView().RawPixelsPerViewPixel;
    splashProgressRing.SetValue(Canvas.LeftProperty, (splashImageRect.Width / density) \* 0.5 - splashProgressRing.Width \* 0.5);
    splashProgressRing.SetValue(Canvas.TopProperty, (splashImageRect.Height/density)\*0.5 +(splashImageRect.Height/density)\*0.2);
}
}
```
目前为止，也就这么多了，以后发现问题了再更新~