---
title: Win10 UWP使用DES加密解密字符串（自定义密码）
tags:
  - UWP
categories:
  - - 开发笔记
date: 2017-12-17 02:51:32
---

在我们开发应用时，一些重要的数据如果以明文的方式保存，自然是很不安全的。因此，我就需要将所保存的账户信息加密以后再保存。微软的官方文档中有这方面的内容，但是采用的是生成随机密钥的形式。我需要的是采用用户自己设置的密码加密和解密内容。一直不知道改怎么实现（个人水平太差），在看到周老师的[【WP开发】加密篇：双向加密](http://www.cnblogs.com/tcjiaan/p/4303918.html)的时候才知道了具体的操作方法。废话不多说了，直接上最终完成效果。
微软的官方文档：[https://msdn.microsoft.com/library/windows/apps/br241537 ](https://msdn.microsoft.com/library/windows/apps/br241537) [https://docs.microsoft.com/zh-cn/windows/uwp/security/cryptographic-keys](https://docs.microsoft.com/zh-cn/windows/uwp/security/cryptographic-keys) [![](http://www.yzj0308.com/wp-content/uploads/2017/12/Des-SampleDemo--1024x765.gif)](http://www.yzj0308.com/wp-content/uploads/2017/12/Des-SampleDemo-.gif) 加密需要Key和IV两个值，他们都是数组的形式Byte\[\]。我写成了一个类，方便调用。
```c#
 using System; 
 using System.Collections.Generic; 
 using System.Linq; 
 using System.Text; 
 using System.Threading.Tasks; 
 using Windows.Security.Cryptography; 
 using Windows.Security.Cryptography.Core; 
 using Windows.Storage.Streams; 
 namespace Des-Sample 
 { 
   public class DESCBCHelper 
   { 
     public static string EnPassword(string value)
      { 
        string _password = AppSettingsHelper.GetSettings("password").ToString(); 
        value = DESCECEncryption(value, _password, _password); return value; 
      } 

      public static string DePassword(string value) { 
        string _password = AppSettingsHelper.GetSettings("password").ToString(); 
        value = DESCBCDecryption(value, _password, _password); 
        return value; 
      } 
      //加密数据
      private static string DESCECEncryption(string strMsg,string iv,string key) 
      { 
        //处理strMsg,不能为空，所以加一个空格。 
        if (strMsg == "") { strMsg = " "; } 
        //加密模式，选择带Pkcs7的编码方式会自动补全所缺的字节，否则加密内容必须是8字节的倍数。 
        String strAlgName = SymmetricAlgorithmNames.DesCbcPkcs7; 
        //把要加密的字符串进行UTF8编码并写入缓冲区
        IBuffer buffMsg = CryptographicBuffer.ConvertStringToBinary(strMsg, BinaryStringEncoding.Utf8); SymmetricKeyAlgorithmProvider objAlg = SymmetricKeyAlgorithmProvider.OpenAlgorithm(strAlgName); 
        //处理key 
        if (key.Length < 8) 
        { 
          key = key + " "; 
        } 
        byte\[\] byteKey = Encoding.UTF8.GetBytes(key); 
        IBuffer buffKey = CryptographicBuffer.CreateFromByteArray(byteKey); CryptographicKey cryKey = objAlg.CreateSymmetricKey(buffKey); 
        //处理iv，iv必须大于8 
        if (iv.Length < 8) 
        { 
          iv = iv + " "; 
        } 
        byte[] byteIV = Encoding.UTF8.GetBytes(iv); IBuffer buffIV = CryptographicBuffer.CreateFromByteArray(byteIV); 
        //加密字符串 
        IBuffer buffEncrypt = CryptographicEngine.Encrypt(cryKey, buffMsg,buffIV); 
        //将缓冲区已加密的字符串转换为
        Base64 string base64Msg = CryptographicBuffer.EncodeToBase64String(buffEncrypt); return base64Msg; 
      } 
      //解密数据 
      private static string DESCBCDecryption(string strMsg, string iv, string key) 
      { 
        //加密模式 
        String strAlgName = SymmetricAlgorithmNames.DesCbcPkcs7; SymmetricKeyAlgorithmProvider objAlg = SymmetricKeyAlgorithmProvider.OpenAlgorithm(strAlgName); 
        //把Base64转换为string，并写入缓冲区 
        IBuffer buffMsg = CryptographicBuffer.DecodeFromBase64String(strMsg); 
        //处理key 
        if (key.Length < 8) { key = key + " "; } 
        byte[] byteKey = Encoding.UTF8.GetBytes(key); IBuffer buffKey = CryptographicBuffer.CreateFromByteArray(byteKey); CryptographicKey cryKey = objAlg.CreateSymmetricKey(buffKey); 
        //处理iv //iv必须大于8 
        if (iv.Length < 8) { iv = iv + " "; } 
        byte[] byteIV = Encoding.UTF8.GetBytes(iv); IBuffer buffIV = CryptographicBuffer.CreateFromByteArray(byteIV); 
        //解密 
        IBuffer buffDecrypted = CryptographicEngine.Decrypt(cryKey, buffMsg, buffIV); string result = CryptographicBuffer.ConvertBinaryToString(BinaryStringEncoding.Utf8, buffDecrypted); return result; 
        } 
    } 
} 
```
其中：AppSettingsHelper是我写的一个存储本地设置的帮助类。 需要注意的点：

- 加密用的密码Key和IV长度都必须大于8，否则无法加密。我的处理方法不具有通用性（我获取的密码是大于等于6位的），或者干脆可以限制密码长度不小于8.
- 需要加密的内容不能为空。
- 为了演示方便我把key和Iv用了相同的值，实际操作中不要使用相同的值，增加安全性。

调用方法： 
```c#
//加密
string result = DESCBCHelper.EnPassword(tb1.Text);  
//解密
string result = DESCBCHelper.DePassword(resultTB.Text);
```
DEMO下载： 

OneDrive链接：[Des-Sample](https://1drv.ms/u/s!ApMNS_xXtxUkmcQehBskSO2iq1aFeg) 

百度云链接：[Des-Sample]("http://pan.baidu.com/s/1NjSo-1ikbBD7LSvEzVc6tA) 密码：xkgw"