---
title: Win10 UWP 获取屏幕分辨率
tags:
  - UWP
categories:
  - - 开发笔记
date: 2019-06-13 15:46:08
---

获取屏幕真实的分辨率。
```c#
using Windows.Graphics.Display;

var displayInformation = DisplayInformation.GetForCurrentView();
var screenSize = new Size(displayInformation.ScreenWidthInRawPixels, 
                          displayInformation.ScreenHeightInRawPixels);
```