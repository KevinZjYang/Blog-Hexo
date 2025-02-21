---
title: 反编译安卓apk文件
tags:
  - 安卓
categories:
  - - 开发笔记
date: 2020-01-12 02:48:00
---

需要用到的工具：

*   [apktool](https://ibotpeaches.github.io/Apktool/install/)
*   [jd-gui](https://github.com/java-decompiler/jd-gui)
*   [dex2jar](https://github.com/pxb1988/dex2jar)

## 方法一

使用apktool，下载解压apktool，然后打开cmd，进入apktool所在的文件夹，命令 `apktool d Demo.apk`

## 方法二

下载jd-gui和dex2jar，全部解压，解压Demo.apk，复制里面的.dex文件到dex2jar的文件夹内， 然后打开cmd，进入dex2jar所在的文件夹，命令`d2j-dex2jar classes.dex` 。会在文件夹中生成 classes-dex2jar.jar。

打开jd-gui，把上面生成的 classes-dex2jar.jar 打开。就可以看到java源码了。

参考文章： [https://juejin.im/post/5d804928f265da03a7160717](https://juejin.im/post/5d804928f265da03a7160717)