---
title: Win10 UWP播放视频时阻止屏幕锁定
tags:
  - UWP
categories:
  - - 开发笔记
date: 2019-01-25 16:20:15
---

在全屏播放视频时，设备屏幕会自动保持常亮。但是在非全屏状态下，屏幕就会自动关闭。所以我们需要让屏幕在播放状态时，保持常亮。

微软的官方文档里面是有详细的介绍。[微软官方文档链接](https://docs.microsoft.com/zh-cn/windows/uwp/design/controls-and-patterns/media-playback)  
但是完全按照这个代码写会报错。也是真真的醉了～?

![](https://gysbda.sn.files.1drv.com/y4mI_fPYfl2_Ro74tI1MxQ5kscY7zTkcisHtZG82Miqd13VmsS_gvwqqTbt3ahEycBZ1HB3dFK4JE6MlfFw1SOdp96DqJhBBb2avd1tb0NWM-7vDkwBjaaa6TI1JbEuePiu3_2kky5NTe6W8VxyVo1lEJpWvle4B0xZ74Am52L3hU2Hv7NcvMB7S7iSMIFMcRFTJmMTQOCz73UyhT-pZiYtIQ?width=1002&height=534&cropmode=none)

解决方法：（按照如下代码就没问题了）

```
 private async void PlaybackSession_PlaybackStateChanged(MediaPlaybackSession sender, object args)
        {
            if (sender is MediaPlaybackSession playbackSession && playbackSession.NaturalVideoHeight != 0)
            {
                if (playbackSession.PlaybackState == MediaPlaybackState.Playing)
                {
　　　　　　　　　　　// 跨线程操作UI　
                    await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
                    {
                        if (appDisplayRequest == null)
                        {
                            // This call creates an instance of the DisplayRequest object
                            appDisplayRequest = new DisplayRequest();
                            appDisplayRequest.RequestActive();
                        }
                    });
                    
                }
                else // PlaybackState is Buffering, None, Opening, or Paused.
                {
                    await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
                    {
                        if (appDisplayRequest != null)
                        {
                            // Deactivate the display request and set the var to null.
                            appDisplayRequest.RequestRelease();
                            appDisplayRequest = null;
                        }
                    });
                    
                }
            }
        }
```

话说，１６年微软官方博客写这个的时候下面就有人评论这个问题。官方文档对我这样的小白也太不友好了。?  
链接：[How to prevent screen locks in your UWP apps](https://blogs.windows.com/buildingapps/2016/05/24/how-to-prevent-screen-locks-in-your-uwp-apps/)