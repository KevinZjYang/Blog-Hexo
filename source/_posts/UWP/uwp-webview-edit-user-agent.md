---
title: UWP WebView修改User-Agent
tags:
  - UWP
  - WebView
categories:
  - - 开发笔记
date: 2020-01-01 22:56:41
---

这个需求源于开发[开眼UWP](http://t.cn/E5CXWi4)的时候，有一个[网页](https://www.eyepetizer.net/campaign/rewind2019.html?shareable=true#1)无法正常播放音频和视频。有小伙伴说可能是因为网页限制了只能在手机端使用（其实不是）。所以找到了以下方法。

## 解决方法

方法出处：[Setting a custom User-Agent in the UWP WebView control](https://www.pedrolamas.com/2017/03/21/setting-a-custom-user-agent-in-the-uwp-webview-control/)

首先写了一个类，导入了“urlmon.dll”这个win32的api。 [Win32 APIs在UWP中可用的列表。](https://docs.microsoft.com/en-us/uwp/win32-and-com/win32-apis)

 public class UserAgentHelper
    {
        \[DllImport("urlmon.dll", CharSet = CharSet.Ansi, ExactSpelling = true)\]
        private static extern int UrlMkSetSessionOption(int dwOption, string pBuffer, int dwBufferLength, int dwReserved);

        private const int URLMON\_OPTION\_USERAGENT = 0x10000001;

        public static void SetDefaultUserAgent(string userAgent)
        {
            UrlMkSetSessionOption(URLMON\_OPTION\_USERAGENT, userAgent, userAgent.Length, 0);
        }
    }

然后在App.cs

 public App()
        {
            InitializeComponent();

            UserAgentHelper.SetDefaultUserAgent("Mozilla/5.0 (Linux; U; Android 10; zh-cn; MIX 2S Build/QKQ1.190828.002) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/71.0.3578.141 Mobile Safari/537.36 XiaoMi/MiuiBrowser/11.4.15");

        }

就这样，应用中所有的WebView都修改了User-Agent。不用单独设置。

## 顺便提一下

在网上搜索修改UA出现最多的是通过修改`NavigateWithHttpRequestMessage`来修改。当时我试了没有效果，不知道什么原因。下面把我的代码放上来，希望那天有大佬看到了，指导一下。

首先订阅`NavigationStarting`事件

\_webView.NavigationStarting += \_webView\_NavigationStarting;

然后在 `NavigationStarting` 事件中取消事件，相当于拦截了webview的加载，然后在Header中添加UA。

  private void \_webView\_NavigationStarting(WebView sender, WebViewNavigationStartingEventArgs args)
        {
            \_webView.NavigationStarting -= \_webView\_NavigationStarting;
            args.Cancel = true;
            AddUserAgent(args.Uri);
        }

private void AddUserAgent(Uri uri)
        {
            var httpRequestMessage = new HttpRequestMessage(HttpMethod.Get, uri);
            var userAgent = "MQQBrowser/26 Mozilla/5.0 (Linux; U; Android 2.3.7; zh-cn; MB200 Build/GRJ22; CyanogenMod-7) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1";
            httpRequestMessage.Headers.Add("User-Agent", userAgent);
            \_webView.NavigateWithHttpRequestMessage(httpRequestMessage);
        }