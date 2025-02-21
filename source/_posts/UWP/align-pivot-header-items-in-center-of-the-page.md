---
title: 让Pivot Headers在页面居中
tags:
  - UWP
categories:
  - - 开发笔记
date: 2020-01-03 19:01:24
---

想让所有的Header都在页面居中，于是找到了以下方法。 [找到答案的页面](https://stackoverflow.com/questions/37391884/align-pivot-header-items-in-center-of-the-page)

通过修改Pivot的模板中的下面内容。其实就是在模板中搜索“HeaderClipper”，然后把`HorizontalContentAlignment="Stretch"`改为`HorizontalContentAlignment="Center"`。大功告成。

```
<!-- some code before -->
<ContentControl x:Name="HeaderClipper" Grid.Column="1" HorizontalContentAlignment="Center" UseSystemFocusVisuals="True">
<!-- some code before -->
```