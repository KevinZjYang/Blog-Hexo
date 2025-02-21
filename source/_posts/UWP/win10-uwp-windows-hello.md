---
title: Win10 UWP使用Windows Hello登录应用
tags:
  - UWP
categories:
  - - 开发笔记
date: 2018-04-10 11:48:10
---

Win10 UWP使用Windows Hello登录应用。Windows Hello是一种生物特征授权方式，让你可以实时访问自己的Windows10设备。有了Windows Hello，用户只需要露一下脸，动动手指，就能立刻被运行Windows10的新设备所识别。 Windows Hello不仅比输入密码更加方便，也更加安全。系统能够让你无需存储在本地或网络服务器的密码即可进行认证应用、企业内容甚至包括某些在线体验。（引用自[百度百科](https://baike.baidu.com/item/windows%20hello/16961229?fr=aladdin)）

让自己的UWP应用使用Windows Hello是一件很酷的事情。原本以为会是很难的一件事，没想到很简单就实现了。本文仅实现了本地登录。（[按照微软官方文档实现](https://docs.microsoft.com/zh-cn/windows/uwp/security/microsoft-passport-login)）。

C#,记得引用`using Windows.Security.Credentials`命名空间

第一步，验证设备是否可以使用Windows Hello。需要满足两个条件：

1.  有一个登录的微软账户；
2.  已经设置PIN登录或者surface系列的虹膜识别登录，或者其他生物识别设备都是可用的。

用下面的代码实现。

```c#
public static async Task<bool> MicrosoftPassportAvailableCheckAsync()
{
    bool keyCredentialAvailable = await KeyCredentialManager.IsSupportedAsync();
    if (keyCredentialAvailable == false)
    {
        // Key credential is not enabled yet as user
        // needs to connect to a Microsoft Account and select a PIN in the connecting flow.
        Debug.WriteLine("Microsoft Passport is not setup!\nPlease go to Windows Settings and set up a PIN to use it.");
        return false;
    }
    return true;
}
```

使用时这样调用：

```c#
bool value = await MicrosoftPassportHelper.MicrosoftPassportAvailableCheckAsync()
```

返回是“true”说明可用。

第二步，在本地创建登录的密钥。

```c#
public static async Task<bool> CreatePassportKeyAsync(string account)
{
    KeyCredentialRetrievalResult keyCreationResult = await KeyCredentialManager.RequestCreateAsync(accountId, KeyCredentialCreationOption.ReplaceExisting);
    switch (keyCreationResult.Status)
    {
        case KeyCredentialStatus.Success:
            Debug.WriteLine("Successfully made key");
            return true;
        case KeyCredentialStatus.UserCanceled:
            Debug.WriteLine("User cancelled sign-in process.");
            break;
        case KeyCredentialStatus.NotFound:
            // User needs to setup Microsoft Passport
            Debug.WriteLine("Microsoft Passport is not setup!\nPlease go to Windows Settings and set up a PIN to use it.");
            break;
        default:
            break;
    }
    return false;
}
```

同样的使用方法，返回“true”说明注册成功。account使用你的App名称比较好。

第三步，登录应用，这个有两个过程：

1.  获取已经注册的密钥，如果存在调用安全验证，输入PIN或者其他生物识别特征（虹膜，指纹等等）登录。
2.  如果不存在调用注册密钥过程。重新注册。

第一个过程代码如下：

```c#
public static async Task<bool> GetPassportAuthenticationMessageAsync(string account)
   {
       KeyCredentialRetrievalResult openKeyResult = await KeyCredentialManager.OpenAsync(account);
       if (openKeyResult.Status == KeyCredentialStatus.Success)
       {
           return true;
       }
       else if (openKeyResult.Status == KeyCredentialStatus.NotFound)
       {
           if (await CreatePassportKeyAsync(account))
           {
               // If the Passport Key was again successfully created, Microsoft Passport has just been reset.
               // Now that the Passport Key has been reset for the _account retry sign in.
               return await GetPassportAuthenticationMessageAsync(account);
           }
       }
       // Can't use Passport right now, try again later
       return false;
   }
```

第二个过程：调用了安全验证，枚举了所有的可能性，在这里我把其他的情况都返回“false”，只有验证成功的情况返回“true”。

```c#
public static async Task<bool> ConsentVerifierAsync()
   {
       var consentResult = await UserConsentVerifier.RequestVerificationAsync(HelloAccount);
       switch (consentResult)
       {
           case UserConsentVerificationResult.Verified:
               return true;
           case UserConsentVerificationResult.DeviceNotPresent:
               return false;
           case UserConsentVerificationResult.NotConfiguredForUser:
               return false;
           case UserConsentVerificationResult.DisabledByPolicy:
               return false;
           case UserConsentVerificationResult.DeviceBusy:
               return false;
           case UserConsentVerificationResult.RetriesExhausted:
               return false;
           case UserConsentVerificationResult.Canceled:
               return false;
           default:
               break;
       }
       return false;
   }
```

可以这样使用，验证成功后导航到指定页面，代码如下：

```c#
if (await MicrosoftPassportHelper.ConsentVerifierAsync())
{
    Frame.Navigate(typeof(MainPage));
}
```

这样就完成了一个登录的整个过程。

注销所注册的密钥的代码如下：

```c#
public static async void RemovePassportAccountAsync(string account)
{    
// Open the account with Passport    
KeyCredentialRetrievalResult keyOpenResult = await KeyCredentialManager.OpenAsync(account);    
// Then delete the account from the machines list of Passport Accounts    await KeyCredentialManager.DeleteAsync(account);}
```

![](/source/post_img/Demo.gif)

如果需要使用Windows Hello 登录服务，可以去看看[微软的官方文档](https://docs.microsoft.com/zh-cn/windows/uwp/security/microsoft-passport-login-auth-service)。