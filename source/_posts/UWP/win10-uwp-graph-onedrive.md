---
title: Win 10 UWP使用Microsoft Graph上传和下载OneDrive文件（一）
tags:
  - UWP
categories:
  - - 开发笔记
date: 2018-05-28 05:03:42
---

在UWP应用中如何使用Microsoft Graph上传和下载OneDrive文件？使用Graph可以调用微软的office，OneDrive，联系人，邮件，日历等很多的接口，因此使用Graph是非常方便的一种做法来开发win10 UWP应用。这是我开发过程中使用OneDrive的方法，记录下来。本篇先讲述如何使用Microsoft Graph登陆账户。
<!-- more -->
一、首先，使用Microsoft Graph，需要安装两个Nuget包。

> Microsoft.Graph; Microsoft.Identity.Client;(搜索时需要勾选“包括预发行版”，否则找不到）

  二、注册ClientID（[apps.dev.microsoft.com](http://apps.dev.microsoft.com)）

1.  打开上面地址后，登陆后，点击“添加应用”
2.  创建应用后，获取你自己的“应用程序ID”和“自定义重定向URL"。（不要使用我举例的这个，需要获取自己的）

“应用程序Id”：9a032f4d-b5bb-49f7-8156-fb387b43ff94 “自定义重定向URL"：msal9a032f4d-b5bb-49f7-8156-fb387b43ff94://auth 三、登陆和退出 1.我将获取令牌和退出写成了一个类（MicrosoftGraphHelper），便于调用。
```c#
using Microsoft.Graph;
using Microsoft.Identity.Client;
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;
namespace Microsoft\_Graph\_Demo
{
    public class MicrosoftGraphHelper
    {
        static string ClientId = "9a032f4d-b5bb-49f7-8156-fb387b43ff94";
        public static string\[\] Scopes = { "User.Read" }; // 设置权限，登陆账户读取
        public static PublicClientApplication IdentityClientApp = new PublicClientApplication(ClientId);
        public static string TokenForUser = null;
        public static DateTimeOffset Expiration;
        private static GraphServiceClient graphClient = null;
        public static GraphServiceClient GetAuthenticatedClient()
        {
            if (graphClient == null)
            {
                // 创建 Microsoft Graph client.
                try
                {
                    graphClient = new GraphServiceClient(
                    "https://graph.microsoft.com/v1.0",
                    new DelegateAuthenticationProvider(
                    async (requestMessage) =>
                    {
                        var token = await GetTokenForUserAsync();
                        requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);
                    }));
                    return graphClient;
                }
                catch (Exception ex)
                {
                    Debug.WriteLine("Could not create a graph client: " + ex.Message);
                }
            }
            return graphClient;
        }
        // 获取用户标记
        public static async Task GetTokenForUserAsync()
        {
            AuthenticationResult authResult;
            try
            {
                authResult = await IdentityClientApp.AcquireTokenSilentAsync(Scopes, IdentityClientApp.Users.First());
                TokenForUser = authResult.AccessToken;
            }
            catch (Exception)
            {
                if (TokenForUser == null  Expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
                {
                    authResult = await IdentityClientApp.AcquireTokenAsync(Scopes);
                    TokenForUser = authResult.AccessToken;
                    Expiration = authResult.ExpiresOn;
                }
            }
            return TokenForUser;
        }
        // 退出登陆
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                IdentityClientApp.Remove(user);
            }
            graphClient = null;
            TokenForUser = null;
        }
    }
}
```
2.登陆，并获取登陆账户信息 MainPage.xaml
```xml
<button name="LoginButton"></button>
<button name="SignOutButton"></button>
```
```c#
MainPage.xaml.cs

    private async void LoginButton\_Click(object sender, RoutedEventArgs e)
    {
        try
        {
            var graphClient = MicrosoftGraphHelper.GetAuthenticatedClient();
            if (graphClient != null)
            {
                var user = await graphClient.Me.Request().GetAsync(); 
                // 获取登陆账户的信息 
                ResultTextBlock.Text = user.DisplayName + "-" + user.UserPrincipalName; }
            else { ResultTextBlock.Text = "GraphClient = null"; } }
        catch (MsalException) { ResultTextBlock.Text = "Login Failed"; } }
    private void SignOutButton\_Click(object sender, RoutedEventArgs e)
    {
        try
        {
            MicrosoftGraphHelper.SignOut();
            ResultTextBlock.Text = "Sign Out Success"; }
        catch (Exception)
        {
            ResultTextBlock.Text = "Sign Out Failed";
        }
    }
```
以上就完成了Microsoft Graph的登陆，效果如下：
![](Demo.gif)

使用Microsoft Graph上传和下载文件到OneDrive将在下一篇文章中介绍。

本文Demo: 

OneDrive: 链接：[Microsoft-Graph-Demo.rar](https://1drv.ms/u/s!ApMNS_xXtxUkmcQdol_pRtT4S_QBDA) 

百度云：链接：[Microsoft-Graph-Demo.rar](https://pan.baidu.com/s/1Sfn6iaFYN12RAhQBLB0DcA) 密码：m5kp