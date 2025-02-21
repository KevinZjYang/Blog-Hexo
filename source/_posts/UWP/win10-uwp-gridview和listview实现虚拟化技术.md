---
title: Win10 UWP GridView和ListView列表虚拟化问题
tags:
  - GridView
  - ListView
  - UWP
  - 虚拟化
categories:
  - - 开发笔记
date: 2018-11-07 13:02:43
---

当我们使用GridView和ListView加载大量数据的时候，为了提高列表的效率，就会用到虚拟化技术。虚拟化技术会计算可见项的数量，然后只为可见项创建UI。GridView和ListView是已经实现虚拟化的。但是为什么有时候没效果呢？看看下文也许对你有所帮助。
<!-- more -->
## 一、数据虚拟化结构

ListView和GridView是支持虚拟化的。其他控件只要实现了虚拟化面板（ItemsWarpGrid或ItemsStackPanel）就可以支持虚拟化。但是要注意以下情况，否则虚拟化会失效。

### ListView外面嵌套ScrollViewer

```xml
<ScrollViewer>
    <ListView/>
</ScrollViewer>
```

像上面这样数据的虚拟化就会失效（其实很多人外面套ScrollViewer是为了实现增量加载）。为什么呢，因为ListView虚拟化结构其实是这样的。

```xml
<ListView >
    <ListView.Template>
         <ControlTemplate>
              <ScrollViewer>
                  <ItemsPresenter />
              </ScrollViewer>
          </ControlTemplate>
     </ListView.Template>
 </ListView>
```

那如果我非要在外面套一个ScrollViewer怎么实现虚拟化呢？可以像下面这样做。去掉ItemsPresenter外面的ScrollViewer.

```xml
<ScrollViewer>
   <ListView >
    <ListView.Template>
         <ControlTemplate>
              <ItemsPresenter />
          </ControlTemplate>
     </ListView.Template>
   </ListView>
</ScrollViewer>
```

## 二、一些说明

**例子1**：做简单的数据加载的时候，比如下面这个例子（滚动到底部自动加载数据,后记：其实到底部加载数据其实有更优雅的方式，外面套ScrollViewer太土狗了）。这个其实破坏了ListView原有的虚拟化结构。

```xml
   <ScrollViewer x:Name="ScrollRoot"
                  ViewChanged="ScrollRoot_ViewChanged"
                  ScrollViewer.VerticalScrollMode="Auto">
        <ListView SelectionMode="None"
                  IsItemClickEnabled="True"
                  ItemsSource="{Binding Allperson}"
                  ItemTemplate="{StaticResource NumberTemplate}">
            <ListView.Template>
                <ControlTemplate>
                    <ItemsPresenter />
                </ControlTemplate>
            </ListView.Template>
        </ListView>
    </ScrollViewer>
```

![](https://i9jcsq.sn.files.1drv.com/y4mQuDaen0GoZLmja0jJAKb3r2Qm6gN9fsbmr8ZwA0V9cWPV0pdbkoFsOorUKu5LVHXSWFoYcHHmkrNnEJmz-8weWyBbNLNhmvRcxIMv0zYcO1W9ZScRTIXYPtQJ_dSmgZylNZOymc_T9jp-tLNg9PPqEZBM8HRdAiJq8-Wq0x0Oqd1Uoiq_ZhMvsz1kfNtBqTdHw0eyWVT_Ev2jNYyfGrL-g?width=660&height=516&cropmode=none)

开启数据虚拟化，首次加载10000个数据，到底部后每次加载1000个，实时可视化树和内存占用如下图所示：

![](https://31pseq.sn.files.1drv.com/y4m2wI4QrSMIhaMh9jyFWEFpSn6EdKRTcygK9IDWz1zSkLRsKhTvBb4hw166PYVj9eThtss6ZLnmdKAHbe8h65_80HcAQqtFUi8UGM6H974g2Ia07HUp1bk4MZT1P02BJQicYEEdHat2Oq1E7XZEcaC_T4y12xDtG67Hx-ZiK4ZEmRq7nmAEdrZ7zgQ3u0R61qlhIboq4FTMI5N5Vka3C2JcQ?width=326&height=298&cropmode=none)

![](https://5qgxpg.sn.files.1drv.com/y4mTNSmQ5S-Ky0S_f7ZR-I131vok9q-3doXLrhfzLAsI4YC84_vVw_6hnjWvGCkLRtocaguCKPF4wWkLKgr3DGs7THnBLU6ko3yJqXiXDHr-y6PDoaP93u3wnkhjCDf6KlUFGK-QQExkFBwQ9-Jlbdblw5K7gHCIWiRD_k3BzV5HIYzmqbILJxCii3msaWPJ9s5GqQ7cyL5_vUvukJuJF9hbw?width=344&height=562&cropmode=none)

上面的例子很明显虚拟化是生效的。但是我们实际做项目的时候，数据模型可不会这么简单。这时候我们就会发现实时可视化树的数量一直在增加，内存也一直在上涨。似乎虚拟化是失效的。那到底怎样呢？看看下个例子。

**例子2**：**[开眼UWP](http://https://www.microsoft.com/store/apps/9NPG6WJ98L88)** 的首页推荐页是一个ListView，ListView里存在多个不同的模板。有的模板里还嵌套着GridView，GridView里还有两个不同的模板。

[![](https://f0thpg.dm.files.1drv.com/y4mPwyjF0slBqmFrB_t1fuzAZV7s2oUyMyAgTHa6eGZCA2ueODUKZtpeHTLbr_T3hSe_RCnv9w3usxVTtnymtoh_H_EmtDoruK1KxFpch76IrARvRqvSAqU2BzexFsRoj9ndNcDnRb7bTAF64_0niihUnG3xOGYwBffeQvza3kYI_a-rfegtriGpAKTO3MJ4jyefTurFuYUvHa_w2H_urU9cw?width=2736&height=1824&cropmode=none)](https://grzz4q.sn.files.1drv.com/y4mTZzZlRn94B0o1MarjjRP8dypypCFSuEsiTbxbZkAMp9UnWuyugzsAaVQMdHxl9TI3e8HnHIzUyCjeFKp9f6WUPaCXRKcB3RvVElh7_6DdlzksRippGE5mWN96QCA8wz_hNpllk8T5lfzZ86esxndubvi2UI-Cu8nj6KgWzWvDzHu3WKCZcqo9qTv5hZBPBU5oK83ZBmBX9H1YTpXFJFyzQ?width=1024&height=676&cropmode=none)

这个问题就比较棘手，这个页面虽然虚拟化是正常的。但是内存一直很高。达到了将近500MB（一直上下滚动）。直到我看到了[官方文档](https://docs.microsoft.com/zh-cn/windows/uwp/debug-test-perf/optimize-gridview-and-listview#container-recycling-with-heterogeneous-collections)的下面文字：

>   
> 项目模板选择器 ([**DataTemplateSelector**](https://msdn.microsoft.com/library/windows/apps/BR209469)) 允许应用根据要显示的数据项目类型在运行时返回不同的项目模板。 这使开发更高效，但也使 UI 虚拟化更困难，因为不是每一个项目模板都可重复用于每个数据项目。  
> 在循环使用项目 (**ListViewItem**/**GridViewItem**) 时，框架必须确定可用于循环队列（循环队列是当前未用于显示数据的项目的缓存）的项目是否具有可与当前数据项目所需的项目模板匹配的项目模板。 如果循环队列中没有带有相应项目模板的项目，则将创建一个新项目，并为其实例化相应的项目模板。 另一方面，如果循环队列包含带有相应项目模板的项目，则该项目将从循环队列中删除并用于当前数据项目。 在仅使用少量项目模板以及在使用不同项目模板的项目集锦分布均匀的情况下，项目模板选择器有效。  
> 当使用不同项目模板的项目分布不均匀时，可能需要在平移期间创建新项目模板，并且这将取消虚拟化所提供的许多好处。 此外，在评估是否可为当前数据项目重复使用特定的容器时，项目模板选择器仅考虑五个可能的候选项。 因此，在应用中使用项目模板选择器前，你应谨慎考虑你的数据是否适用于项目模板选择器。 如果你的集锦大部分是同类，则选择器将在大多数时间（可能是所有时间）返回相同的类型。 请注意你要为上述罕见的同质例外所付出的代价，并考虑是否首选使用 [**ChoosingItemContainer**](https://msdn.microsoft.com/library/windows/apps/windows.ui.xaml.controls.listviewbase.choosingitemcontainer)（或两个项目控件）。

ChoosingItemContainer目前我还不会使用。等学会了看看是否可以解决这个问题。

**例子3**：在[爱美图](https://www.microsoft.com/store/apps/9NJLDD2934RK)中，是一个GridView增量加载数据，在虚拟化被破坏时，内存一直上涨，甚至超过1GB。但是修复虚拟化以后。情况如下：

![](https://es8tta.sn.files.1drv.com/y4mQYcSGjN8vRPn4DM0lq2ESxci3S3zcCPbVT7cGtCvYSfXfMS8ej3H6b9gnFpU6LjMEyxX9XoygUUNndw-H_RJKia4hNCBBE75aKfTj_FLjvzRV73BFVSJpKKQSTC9Lhn4ouToMajCjFxapILa1ZQVsgjHZHWvRikxar2EvIuvmlUlQbZdLo-9FLFPWZuZIlO6c2AaMI4r2Jf9H2QWNcoVuw?width=1003&height=792&cropmode=none)

![](https://sy0kna.sn.files.1drv.com/y4m2KFiHy4BfyWQ30315TWtN1xnVDCxBrEJzMvaY5cusT49o5UrlkFHy6S6o7y9-UxZxvL2rSLi5EIQCorvqu3Pyfu5G-i8MNsURZXGtdHPACH-yTSNAWE-p7nyHPtd2QMhfN7ESxJheDhna8sMCuim77NxI2yi7aaSYrZrmzsBLUFRSL0QLDUuNtKDZVWUj4yc2DRy507-WHBXGh0WLbHwJQ?width=329&height=368&cropmode=none)

![](https://hisaha.sn.files.1drv.com/y4m2orZZJ5pS1ZZaQjFWJpmJ7IFfCEJTMDYHa4rD_6iN_uA9K-rR3J4ySh_q1sLFuA_eiYSJk9QWtHX-zNbRleyNB1NtfmUYEv9jqqvEg3UdM2Qh47cdvyLK3Bbdlifha02hK3OhwqWSz87K7B30dhlnJvyW28v4Xy3Xa_vQJosL3e8dBBYHEsmhPI0bKVixrbyfOpQpJDdjK8CR2A3h9qAxg?width=338&height=564&cropmode=none)

这个里面也用了模板选择器，不过就一个广告和图片的选择，比较简单。所以虚拟化的性能没怎么影响。