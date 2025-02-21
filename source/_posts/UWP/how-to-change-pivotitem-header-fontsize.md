---
title: 如何修改PivotItem的Header的字体大小
tags:
  - UWP
categories:
  - - 开发笔记
date: 2019-03-19 16:10:22
---

在开发“[开眼UWP](https://www.microsoft.com/store/apps/9NPG6WJ98L88)”的时候，用到了Piovt，但PivotItem的默认Header字体太大。于是想去修改这个字体的大小。在Piovt的样式文件中找了好久都么找到对应的修改的地方。最后在stackoverflow上找到了下面wp8上用的修改方法。  
[WP8: reduce the pivot page header size?](https://stackoverflow.com/questions/18463588/wp8-reduce-the-pivot-page-header-size)

没想到在UWP上也是通用的。记录方法如下：

```
 <Pivot.HeaderTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding}"
                     FontSize="20" />
        </DataTemplate>
 </Pivot.HeaderTemplate>
```

好水...