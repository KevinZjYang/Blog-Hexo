---
title: Visual Studio一直提示刷新凭据
tags:
  - Visual Studio
categories:
  - - 开发笔记
date: 2019-05-07 03:33:57
---

前几天遇到VS登陆后一直需要重新验证凭据的问题。但是验证的时候会陷入循环登录的过程。即使你不厌其烦的一直登陆。结束后也并没有成功，关闭窗口重新打开，就会发现。又需要验证了。。。

[![循环登陆gif](https://www.yzj0308.com/wp-content/uploads/2019/06/2019061002372347.gif)](https://www.yzj0308.com/visual-studio%e4%b8%80%e7%9b%b4%e6%8f%90%e7%a4%ba%e5%88%b7%e6%96%b0%e5%87%ad%e6%8d%ae/attachment/2019061002372347/)

在折腾搞几天后，终于在网上发现了这篇文章。[救命的文章](https://blog.csdn.net/lindexi_gd/article/details/84251977)

只需要电脑资源管理器，进入下面的路径，删除里面所有的内容即可。重写打开VS,再次登录，一切正常。?

```
%USERPROFILE%\AppData\Local\.IdentityService
```

PS:林更新大佬就是牛逼。