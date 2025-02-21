---
title: Win10 UWP添加定时触发的进程外后台任务
tags:
  - UWP
categories:
  - - 开发笔记
date: 2019-01-04 14:55:09
---

## 序

添加后台任务是开发app，必需的功能。比如动态磁贴就需要后台任务的支持。但是怎么添加后台任务？添加后台任务后为什么不触发？还是有小伙伴遇到这样的问题。所以写下了这篇文章，记录如何快速简单的添加后台任务。

## 一、新建解决方案

1、新建windows通用项目，命名为“BackgroundTaskDemo"

![](https://grze2q.sn.files.1drv.com/y4mEZgNfocMGJFvGrNYC-mMxU3qoqf3bb49kzPBQiy25c_4OxmcC0wkfakRB93HeEh0bVTcNOF1kqCnw7FA9LNXlnmaBD3WHfTuNNOoV5vnsQ7E0pvI6gISXE34ZoqaC0CDHbEvSFoxeaPubzw6AOrN12XxqpuIgMHNxZWcTcj_q1b4c-oDDSkKrqQo4QkGHmgZM1e-UwynOO580D3WVi7c_Q?width=941&height=660&cropmode=none)

2、在解决方案资源管理器，右键解决方案，选择”添加“-”新建项目“，新建windows运行时组件。命名为”BackgroundTasks“，应用内所有的后台任务都写在这里面。

![](https://hisffa.sn.files.1drv.com/y4mMC5Swk-p2vnZzzfh5QA9YnTEIf0Sc5O7wMAWuSy5Fr5o3UqMeZtTwmMn5i_lubcEl3GoRoBzZZAbwJac_Byr7T7dlZ63IEGWLLBqQjCXojyTL37JDcd7CDvfibiGhoTZKr2o03-LgMwQGTeSM8lr_I8HcJyvgRtvgTxe4ex5xv9rOZ42XynEvHfOCFQ0jCMl1Dw-ygDe1pX-lKfF8zcQuQ?width=941&height=660&cropmode=none)

3、删掉默认的class1.cs，新建一个类，命名为“ToastBackgroundTask"

## 二、配置后台任务

1、在”BackgroundTaskDemo"中，打开“Package.appxmanifest”，按下图设置。

![](https://31pxcq.sn.files.1drv.com/y4mwHaLkjVReQZsWB2xb7QbOsb1ush4og3qZJyUpWLQkX-dudOHaPQT3aoQcOngjP1UUL0HxDHeU-WCEeY68eU82GdwIiMQw5omPYY1QGnHHXntBjcrFMY8LJP138oJptlpzkg5jMIfh3I0gXtFp8I21tjHPZsHTVLl1mpDdsy7UWrMAR-c8sWayZ-hSgjIQXX13XHk3Cl7yeebWfSu0AYz8w?width=973&height=751&cropmode=none)

## 三、写后台事件的类

1、由于很多方法在windows运行组件里是不能写的，所以这里单独新建一个类库项目。

![](https://es8yra.sn.files.1drv.com/y4mvxqD3M4YLQitVzPnlDWuAFT0_c3o0jXqH4pAgJuvU6cYg4dZV8dZPqDYelbn7lQaRR1hBQDBmkms9k8ejiI0ovgr9dwvMsHG74dE5ta9N2Ha0DG7wm0_jQOU3HiTcAK3R2w4ymydnvpBxyqD4YX--ZblUfongw6zsd67mUmyREEo_QG8IYurWamqKSZ0YKExl34MU8Zugt_GglJheywRsA?width=942&height=660&cropmode=none)

2、新建一个类，命名为”ToastBackgroundTaskHelper“，这里是做了一个Toast通知。代码如下：（需要安装Nuget：”Microsoft.Toolkit.Uwp.Notifications“）

```c#
using Microsoft.Toolkit.Uwp.Notifications;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Windows.UI.Notifications;

namespace BackgroundTaskShare
{
    public class ToastBackgroundTaskHelper
    {

        public static void PopToast()
        {
            // Generate the toast notification content and pop the toast
            ToastContent content = GenerateToastContent();
            ToastNotificationManager.CreateToastNotifier().Show(new ToastNotification(content.GetXml()));
        }

        public static ToastContent GenerateToastContent()
        {
            return new ToastContent()
            {
                Launch = "action=viewEvent&eventId=1983",
                Scenario = ToastScenario.Reminder,

                Visual = new ToastVisual()
                {
                    BindingGeneric = new ToastBindingGeneric()
                    {
                        Children =
                {
                    new AdaptiveText()
                    {
                        Text = "Adaptive Tiles Meeting"
                    },

                    new AdaptiveText()
                    {
                        Text = "Conf Room 2001 / Building 135"
                    },

                    new AdaptiveText()
                    {
                        Text = "10:00 AM - 10:30 AM"
                    }
                }
                    }
                },

                Actions = new ToastActionsCustom()
                {
                    Inputs =
            {
                new ToastSelectionBox("snoozeTime")
                {
                    DefaultSelectionBoxItemId = "15",
                    Items =
                    {
                        new ToastSelectionBoxItem("1", "1 minute"),
                        new ToastSelectionBoxItem("15", "15 minutes"),
                        new ToastSelectionBoxItem("60", "1 hour"),
                        new ToastSelectionBoxItem("240", "4 hours"),
                        new ToastSelectionBoxItem("1440", "1 day")
                    }
                }
            },

                    Buttons =
            {
                new ToastButtonSnooze()
                {
                    SelectionBoxId = "snoozeTime"
                },

                new ToastButtonDismiss()
            }
                }
            };
        }

    }
}
```

## 四、处理 ToastBackgroundTask

1、要调用 ToastBackgroundTaskHelper 里面的方法，首先需要在BackgroundTasks中添加对BackgroundTaskShare的引用。

![](https://i9jkqq.sn.files.1drv.com/y4mOeNFptIgyJSNCO3Yztx-okWGBzNMwLD2TPjaA0G5bSXNvfByNvGr3us4LbGKgpLalRlp5tiRZbK2zT6Zb-OeiPCiabopQ2QsJpRtKcGIyba3GeN1QgaEyJD1LqRT2mzjrSeD3Q3za8SGdAVr3nCE0I19MBzxg4BksWLQvfNApp3kffFjRsSMAHpSEZBNvJF6X3ocdAaOEScrm3s8dwemAA?width=783&height=550&cropmode=none)

2、处理ToastBackgroundTask.cs，代码如下，

```c#
using Microsoft.Toolkit.Uwp.Notifications;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Windows.ApplicationModel.Background;

namespace BackgroundTasks
{
    // 记得要加sealed
    public sealed class ToastBackgroundTask:IBackgroundTask
    {
        public void Run(IBackgroundTaskInstance taskInstance)
        {
            var defferral = taskInstance.GetDeferral();
            // 这里写你的后台操作，操作耗时如果超过5秒就失败了,有异步操作的记得加async
            BackgroundTaskShare.ToastBackgroundTaskHelper.PopToast();

            defferral.Complete();
        }

       

    }
}
```

## 五、注册和取消后台任务

1、首先在BackgroundTaskDemo中添加对“BackgroundTasks”的引用。

![](https://vkbysw.sn.files.1drv.com/y4mbWOfwMigZNxO5-C5ggEYRbqa2JyPlr1sEXYBqs0sp5sQcVI5-8S7wBBZh8TezfultiHUAda3cHuDACAw24X2wGz4XK7kwKQ3XoBV9uaszU0lYwcp5Io_XrOsERq807hUkoWJC-FJ4iyJYa6EQEo8nqTVuPYTSddgTvBRsWtyC4fKa4y9t9CMAthRmzij_MUdKCVRWT0-qGZevo80aQZ4lA?width=786&height=550&cropmode=none)

2、在 BackgroundTaskDemo中安装Nuget：“Microsoft.Toolkit.Uwp”，使用如下方法进行注册和取消后台任务。

```c#
// 注册 
BackgroundTaskRegistration registered =                    BackgroundTaskHelper.Register(typeof(BackgroundTasks.ToastBackgroundTask),new TimeTrigger(15, false), false, true, new SystemCondition(SystemConditionType.InternetAvailable));

// 取消
BackgroundTaskHelper.Unregister(typeof(BackgroundTasks.ToastBackgroundTask));
```

上面设置为在有网络的情况下每15分钟触发一次后台任务。

## 六、调试后台任务

1、运行Demo后注册后台任务，稍等一会儿，点开VS的生命周期事件，就可以看到ToastBackgroundTask。需要调试的话就可以在 ToastBackgroundTask 中设置断点。

![](https://sy0ola.sn.files.1drv.com/y4mfruaIRS7XasDPB6QBGD-ndEQ1iduTqgvrxQlf0ldx5PYrkMYUeWTUQaHPeRU1YdSGUgwDlPnMRBH9KvHOT_79_efakAK07QTKQvx7SIDXkNcacVM7T0_VHFdQgVKOwhRl-Y-hhK6SDPf0Z1yqIQO7uEw_ks-dlCQ6eQwEAm5SyzKMgmx7LP30A5Xq30s11YvGmpDmXSOxfqFMoIli5-TFw?width=775&height=427&cropmode=none)

2、点击后就会立刻运行后台任务，结果如下。说明代码是没有问题的。

![](https://5qg1ng.sn.files.1drv.com/y4m_bEkZvH08yU2CtNpH67jy6jb4cDqoU6UJ2tLRJBL9KDMvFzVPK6AVW5K5q7waOwRUK5GMBOzCAFBtVt7XA3LzdTDZYsRwsBshuBYe1NIBpz81IP-hTxXe8MvkZuKXcWIYLGblDlaxLoDXQshxe-cPmhz8xHVa8gx1BgvVFoPjkM58Xy_WUX_LZjxS3ccTX3be_MEt_atm2wxC7m_cUFwOw?width=456&height=288&cropmode=none)

这样子就大功告成了。需要的可以下载Demo。

## 七、Demo下载

[BackgroundTaskDemo](https://www.yzj0308.com/wp-content/uploads/2019/01/BackgroundTaskDemo.zip)[Download](https://www.yzj0308.com/wp-content/uploads/2019/01/BackgroundTaskDemo.zip)