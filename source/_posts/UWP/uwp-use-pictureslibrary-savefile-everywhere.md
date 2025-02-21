---
title: UWP 通过KnownFolders.PicturesLibrary任意选择保存文件的位置
tags:
  - UWP
categories:
  - - 开发笔记
date: 2019-08-12 02:06:02
---

发现这个彩蛋，源于我的应用“[爱美图](https://www.microsoft.com/store/apps/9NJLDD2934RK)”，有一个用户在应用下面的评论说：

> 不知道下载到了哪里（后来找到了在OneDrive\\图片\\爱美图），不能自定义下载位置，很不方便

但是看到评论的我一脸懵逼，我使用的是KnownFolders.PicturesLibrary创建的文件夹，按道理不应该在C盘的图片文件夹吗？怎么回到OneDrive里面去了。直到我发现我的台式机也是这样，我又打开surface看了一下，确实在C盘，没错。这就奇怪了。

不过我突然想起来其几天设置过系统自带的照片应用的图片源。这下发现了不得了的事情。

## 情况一

下面这是Surface上照片-设置里源的顺序，**系统C盘的图片在上面，OneDrive在下面**。这种情况下使用“KnownFolders.PicturesLibrary.CreateFolderAsync”获取的文件夹路径在**系统C盘的图片**。

![](https://hcdyxa.sn.files.1drv.com/y4mzkjNsG-tpTvio79mCThbU37CND_pGGIJnaDWDOG1QTDdP0o3TfM8leTFmVMbJ3O5i1ivi2fE5E_3BHTvnWpordTBagaLNrkeWYPD89q1LFby9vRmaqW_eI7w1bIc1-7vWSkQlvovFvO8klhFDAYpmGLG1XvcZUSDZdCWv61AJsGkJfUAALk05ATj0Ce92ooSJCVM3wUzL7WUC6p0N721_A?width=2052&height=1602&cropmode=none)

Surface上的图片-设置

## 情况二

下面这是台式机上照片-设置里源的顺序， **OneDrive** **在上面，** **系统C盘的图片** **在下面**。这种情况下使用“KnownFolders.PicturesLibrary.CreateFolderAsync”获取的文件夹路径在 **OneDrive里的图片文件夹**。

![](https://vivpra.sn.files.1drv.com/y4m3IyimbjuY6ekwvokukg2zZIWsrOB6oIm0nz3mQfOJWJFDRWasE_aHec_NeIkRRqAIxWQcJrOvjhfdNunO3OY_Mua6FGO-JbzXt5NWrXPaUWFxRbkr32QffXdnkrqrN0uB7Jhedd6M5y5r76RKymzWtt0wH-l9wyoEeKVd-xIqVT-EVT6MGsBqeTvY2rBObZuHvCf-2p8TtlNraMFqNNqdw?width=2052&height=1602&cropmode=none)

台式机上图片-设置

## 猜想

这么看来是不是最上面的路径就是 PicturesLibrary 获取到的路径呢？那么我把最上面的路径改成D盘的任意文件夹会怎么样呢？设置如下图。

![](https://yt5mza.sn.files.1drv.com/y4m3LywuxIyH7ydcg6tJ24Np6L0_vz5RoTbvTdHcjbLHxPkIUOcvDF-NdFBTFAcjYSNHM9Vi8edR7b-rPSf7ez0lxKgk7izYGUx-LmQj3E5JU_5dORmy9_XKgmxqpTQ_NMeqIYWRgiywd-ksP2qYMhN3QWTKYvct8hLcmEfeOMqy_h_WF8ZVCCwjMmPFKIm7_CIlNaq8Tut2QAtP00CtD7aEw?width=2052&height=1602&cropmode=none)

然后神奇的事情发生了！！！

![](https://zlpzbg.sn.files.1drv.com/y4mH8I53tG-7Hi_BM0YGK0B6UwP4RDwChn2ani8IllUmZxbD1-X76JxjEnWDBEIxzdYmPo2XIK4Fs8BdY9A__Erf5eC1Z8w2JPndhjFTOCxDd6Adjkyx5GKpOLS4ctVJ83HgwPiQK8jeSaYBMYtRCq1UqjtPogSucjoobH12M3dLNc_sS7sMgF8KKOw5V2EoEpCAqDOygz3dOJxSnkRpcexsg?width=2736&height=1824&cropmode=none)

竟然真的可以，这样其实可以变相的随意设置UWP应用下载图片的位置了！！！

这样来说，那么电影和电视里设置查找视频的源，是不是能达到同样的效果呢？没有测试，有需要可以去试试哦。

PS:大半夜发现了，忍不住想写个文章。。。虽然没人看😂