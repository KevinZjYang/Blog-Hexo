---
title: Microsoft.Services.Store.Engagement was not included in compilation
tags:
  - UWP
categories:
  - - 开发笔记
date: 2019-12-31 01:44:38
---

本文章记录以前遇到的一个和`Microsoft.Services.Store.Engagement`有关的问题。当时解决后没想到最近有小伙伴又遇到了，所以在此做个记录。

## 错误

Method 'StoreServicesEngagementManager.GetDefault()' from assembly 'Microsoft.Services.Store.Engagement' was not included in compilation, but was referenced in xxxxxxx.

描述：只要使用**[Windows Template Studio 3.3](https://github.com/microsoft/WindowsTemplateStudio)** 以前的版本生成了使用`Microsoft.Services.Store.Engagement`的代码，上传商店后就会出现这个问题。本地测试正常。

## 解决方法

来自这个[issue](https://github.com/microsoft/WindowsTemplateStudio/issues/3206)

This is not an issue with using AppCenter. It can affect any app submitted to the store.

**How to fix this in the code in your existing apps**

Manually open the csproj file.

change the following lines from

    <SDKReference Include="Microsoft.Services.Store.Engagement, version=10.0">
      <Name>Microsoft Engagement Framework</Name>
    </SDKReference>
    <SDKReference Include="Microsoft.VCLibs, version=14.0">
      <Name>Visual C++ 2015 Runtime for Universal Windows Platform Apps</Name>
    </SDKReference>

to

    <SDKReference Include="Microsoft.Services.Store.Engagement, Version=10.0">
      <Name>Microsoft Engagement Framework</Name>
    </SDKReference>
    <SDKReference Include="Microsoft.VCLibs, Version=14.0">
      <Name>Visual C++ 2015 Runtime for Universal Windows Platform Apps</Name>
    </SDKReference>

In both cases, it must be `Version` (with a capital 'V') rather than `version`.

其实就是把两个SDK的`version`改成首字母大写的`Version`。