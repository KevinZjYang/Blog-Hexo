---
title: Win10 UWP自定义MediaTransportControl的数据绑定问题
tags:
  - UWP
categories:
  - - 开发笔记
date: 2019-02-14 15:25:26
---

在开发开眼UWP的过程中，需要MediaTransportControl有切换视频播放清晰度的需求。但是过程中发现绑定的数据一直不显示。记录一下过程，写的有点乱。

最后完成的样子：

![](https://56g3lg.sn.files.1drv.com/y4mDALwlWUd9X_4KCczht2NlcXyguB5A_ifvyFIhiCc-__e7OdJ7yuqBK56bkwDZIF5E2pNkmdtIMBt-6N7e4lf29PJi9l2GsflrUcZTRPzR6rROTjtsT3Kt2mcDgk-Ue-f2GeBKG3X85nHMoY_EJaqVUM4pPb_Yzk1PWyv0p840IeJVpTcX7Y0qEAz-ZJwzZinFEftS0zIh9qjJe3ZjvhoLw?width=2736&height=1744&cropmode=none)

自定义MediaTransportControl的方法不说了，详细请看的微软官方文档：[文档链接](https://docs.microsoft.com/zh-cn/windows/uwp/design/controls-and-patterns/custom-transport-controls)

文中提到的文件

*   MediaTransportControlsDictionary.xaml
*   CustomMediaTransportControls.cs
*   PlayVideoControl.xaml
*   PlayVideoControl.xaml.cs

## 一、数据格式
```json
"playInfo": \[{
"height": 270,
"width": 480,
"urlList": \[{
"name": "aliyun",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=low&source=aliyun",
"size": 8142424
},
{
"name": "qcloud",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=low&source=qcloud",
"size": 8142424
},
{
"name": "ucloud",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=low&source=ucloud",
"size": 8142424
}
\],
"name": "流畅",
"type": "low",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=low&source=aliyun"
},
{
"height": 480,
"width": 854,
"urlList": \[{
"name": "aliyun",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=normal&source=aliyun",
"size": 18648494
},
{
"name": "qcloud",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=normal&source=qcloud",
"size": 18648494
},
{
"name": "ucloud",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=normal&source=ucloud",
"size": 18648494
}
\],
"name": "标清",
"type": "normal",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=normal&source=aliyun"
},
{
"height": 720,
"width": 1280,
"urlList": \[{
"name": "aliyun",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=high&source=aliyun",
"size": 30572399
},
{
"name": "qcloud",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=high&source=qcloud",
"size": 30572399
},
{
"name": "ucloud",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=high&source=ucloud",
"size": 30572399
}
\],
"name": "高清",
"type": "high",
"url": "http://baobab.kaiyanapp.com/api/v1/playUrl?vid=148037&resourceType=video&editionType=high&source=aliyun"
}
\]
```
## 二、修改模板，添加按钮

在MediaTransportControlsDictionary.xaml中添加：
```xml
  <!--清晰度切换-->
    <AppBarButton x:Name='PlayInfoButton'
                  Margin="8,0,0,0"
                  Style="{StaticResource AppBarButtonStyle}"
                  Content="{Binding ElementName=PlayInfoListView,Path=SelectedItem.name}">
        <AppBarButton.Flyout>
            <Flyout x:Name="PlayInfoFlyout">
                <ListView x:Name="PlayInfoListView"
                          Width="52"
                          ItemsSource="{Binding Path=PlayInfos,Mode=TwoWay,RelativeSource={RelativeSource TemplatedParent}}"
                          SelectedIndex="{Binding Path=PlayInfo\_SelectedIndex,Mode=TwoWay,RelativeSource={RelativeSource TemplatedParent}}">
                    <ListView.ItemTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding name}" />
                        </DataTemplate>
                    </ListView.ItemTemplate>
                </ListView>
            </Flyout>
        </AppBarButton.Flyout>
    </AppBarButton>
```
这里绑定使用的是Binding,我看别人使用的是TemplateBinding，但是我编译不了?，刚开始自己直接这样ItemsSource="{Binding Path=PlayInfos,Mode=TwoWay}"，发现绑定的数据是空白的。因为页面加载的时候是先初始化MediaTransportControl, 完成后才是页面后台代码的赋值。所以每次初始化的数据都是空白。赋值后数据也不会更新。后来看到一个博客，使用ItemsSource="{Binding Path=PlayInfos,Mode=TwoWay,RelativeSource={RelativeSource TemplatedParent}}"这样就可以更新数据了。  
参考的博客地址：[模板中的 TemplateBinding 问题](https://www.cnblogs.com/hebeiDGL/p/3670667.html)

## 三、定义绑定的依赖项

在CustomMediaTransportControls.cs中添加：
``` c#
// 使用propdp快速生成
 public List<PlayInfo> PlayInfos
 {
     get { return (List<PlayInfo>)GetValue(PlayInfosProperty); }
     set { SetValue(PlayInfosProperty, value); }
 }

// Using a DependencyProperty as the backing store for PlayInfos.  This enables animation, styling, binding, etc...
 public static readonly DependencyProperty PlayInfosProperty =
            DependencyProperty.Register("PlayInfos", typeof(List<PlayInfo>), typeof(CustomMediaTransportControls), new PropertyMetadata(null));
```
## 四、使用

在PlayVideoControl.xaml中添加 :
```xml
 <MediaPlayerElement Name="MainMediaPalyer"
                     AreTransportControlsEnabled="True"
                     AutoPlay="True"
                     DoubleTapped="MainMediaPalyer\_DoubleTapped">
       <MediaPlayerElement.TransportControls>
              <control:CustomMediaTransportControls 
                           x:Name="mediaTransportControl"
                           Style="{StaticResource MyMediaTransportControls}"
                           IsCompact="True"
                           IsCompactOverlayEnabled="True"
                           IsZoomButtonVisible="false"
                           IsCompactOverlayButtonVisible="True"
                           IsVolumeButtonVisible="True"
                           IsStopButtonVisible="False" 
                          PlayInfo\_SelectionChanged="MediaTransportControl\_PlayInfo\_SelectionChanged"/>
        </MediaPlayerElement.TransportControls>
 </MediaPlayerElement>
```
在PlayVideoControl.xaml.cs中赋值：  
mediaTransportControl.PlayInfos = VideoData.playInfo;

## 五、响应清晰度的改变

在CustomMediaTransportControls.cs中的OnApplyTemplate中订阅ListView的SelectionChanged事件。
```c#
public event EventHandler<PlayInfoEventArgs> PlayInfo\_SelectionChanged; 
protected override void OnApplyTemplate()
{
   {
      var playInfoListView = (ListView)GetTemplateChild("PlayInfoListView");
      playInfoListView.SelectionChanged += PlayInfoListView\_SelectionChanged;
    }
    base.OnApplyTemplate();
 }

private void PlayInfoListView\_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
   var playInfoLV = sender as ListView;
   if(playInfoLV.SelectedIndex != -1)
     {
        var info = PlayInfos\[playInfoLV.SelectedIndex\];
        PlayInfo\_SelectionChanged?.Invoke(this, new PlayInfoEventArgs(info));
      }

}

 public class PlayInfoEventArgs : EventArgs
 {

     public PlayInfo Infos { get; set; }
     public PlayInfoEventArgs(PlayInfo info)
     {
            Infos = info; 
     }
 }
```   

然后在PlayVideoControl.xaml.cs页面订阅MediaTransportControl\_PlayInfo\_SelectionChanged事件，更改MediaPlayerElement的Source。
```c#
private void MediaTransportControl\_PlayInfo\_SelectionChanged(object sender, PlayInfoEventArgs e)
{        
   if(playUrl != e.Infos.url)
   {
      MainMediaPalyer.Source = MediaSource.CreateFromUri(new Uri(e.Infos.url));
   }
}
```
## 六、全部代码

### 1、CustomMediaTransportControls.cs
```c#
using EyepetizerTest.Services;
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Data;
using Windows.UI.Xaml.Media;

namespace EyepetizerTest.Controls
{
    public class CustomMediaTransportControls:MediaTransportControls
    {
        public event EventHandler<PlayInfoEventArgs> PlayInfo\_SelectionChanged;
        private bool IsLoading = false;
        public CustomMediaTransportControls()
        {
            this.DefaultStyleKey = typeof(CustomMediaTransportControls);
           
        }

        public string VideoTitle
        {
            get { return (string)GetValue(VideoTitleProperty); }
            set { SetValue(VideoTitleProperty, value); }
        }

        // Using a DependencyProperty as the backing store for VideoTitle.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty VideoTitleProperty =
            DependencyProperty.Register("VideoTitle", typeof(string), typeof(CustomMediaTransportControls), new PropertyMetadata(null));



        public int PlayInfo\_SelectedIndex
        {
            get { return (int)GetValue(PlayInfo\_SelectedIndexProperty); }
            set { SetValue(PlayInfo\_SelectedIndexProperty, value); }
        }

        // Using a DependencyProperty as the backing store for PlayInfo\_SelectedIndex.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty PlayInfo\_SelectedIndexProperty =
            DependencyProperty.Register("PlayInfo\_SelectedIndex", typeof(int), typeof(CustomMediaTransportControls), new PropertyMetadata(-1));


        public List<PlayInfo> PlayInfos
        {
            get { return (List<PlayInfo>)GetValue(PlayInfosProperty); }
            set { SetValue(PlayInfosProperty, value); }
        }

        // Using a DependencyProperty as the backing store for PlayInfos.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty PlayInfosProperty =
            DependencyProperty.Register("PlayInfos", typeof(List<PlayInfo>), typeof(CustomMediaTransportControls), new PropertyMetadata(null));

        protected override void OnApplyTemplate()
        {
            {
                var playInfoListView = (ListView)GetTemplateChild("PlayInfoListView");
                playInfoListView.SelectionChanged += PlayInfoListView\_SelectionChanged;
            }
            {
                var rootGrid = (Grid)GetTemplateChild("RootGrid");
                rootGrid.PointerMoved += RootGrid\_PointerMoved;
            }
            base.OnApplyTemplate();
        }

        private void RootGrid\_PointerMoved(object sender, Windows.UI.Xaml.Input.PointerRoutedEventArgs e)
        {
            if(IsLoading == false)
            {
                IsLoading = true;
                var titlePanelFadeIn = (VisualState)GetTemplateChild("TitlePanelFadeIn");
                titlePanelFadeIn.Storyboard.BeginTime = new TimeSpan(500);
                titlePanelFadeIn.Storyboard.Duration = new Duration(TimeSpan.FromSeconds(4));
                titlePanelFadeIn.Storyboard.Begin();
                titlePanelFadeIn.Storyboard.Completed += Storyboard\_Completed;
            }
                     
        }

        private void Storyboard\_Completed(object sender, object e)
        {
            var titlePanelFadeOut = (VisualState)GetTemplateChild("TitlePanelFadeOut");
            titlePanelFadeOut.Storyboard.BeginTime = new TimeSpan(500);
            titlePanelFadeOut.Storyboard.Begin();
            titlePanelFadeOut.Storyboard.Completed += Storyboard\_Completed1;
        }

        private void Storyboard\_Completed1(object sender, object e)
        {
            IsLoading = false;
        }

        private void PlayInfoListView\_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            var playInfoLV = sender as ListView;
            if(playInfoLV.SelectedIndex != -1)
            {
                var info = PlayInfos\[playInfoLV.SelectedIndex\];
                PlayInfo\_SelectionChanged?.Invoke(this, new PlayInfoEventArgs(info));
            }

        }
       
    }
  
    public class PlayInfoEventArgs : EventArgs
    {

        public PlayInfo Infos { get; set; }
        public PlayInfoEventArgs(PlayInfo info)
        {
            Infos = info; 
        }
    }
    
}
```
### 2、MediaTransportControlsDictionary.xaml
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:local="using:EyepetizerTest.Styles"
                    xmlns:control="using:EyepetizerTest.Controls"
                    xmlns:service="using:EyepetizerTest.Services"
                    xmlns:converter="using:EyepetizerTest.Converters">

    <!-- Default style for MediaTransportControls -->
    <Style TargetType="control:CustomMediaTransportControls"
           x:Key="MyMediaTransportControls">
        <Setter Property="IsTabStop"
                Value="False" />
        <Setter Property="Background"
                Value="Transparent" />
        <Setter Property="FlowDirection"
                Value="LeftToRight" />
        <Setter Property="UseSystemFocusVisuals"
                Value="{StaticResource UseSystemFocusVisuals}" />
        <Setter Property="IsTextScaleFactorEnabled"
                Value="False" />
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="MediaTransportControls">
                    <Grid x:Name="RootGrid"
                          Background="Transparent">

                        <Grid.Resources>
                            <!-- New AppBar button style 48x48 pixels in size -->
                            <Style x:Key="AppBarButtonStyle"
                                   TargetType="AppBarButton"
                                   BasedOn="{StaticResource AppBarButtonRevealStyle}">
                                <Setter Property="Width"
                                        Value="{ThemeResource MTCMediaButtonWidth}" />
                                <Setter Property="Height"
                                        Value="{ThemeResource MTCMediaButtonHeight}" />
                                <Setter Property="AllowFocusOnInteraction"
                                        Value="True" />
                            </Style>
                            <!-- New AppBarToggle button style 48x48 pixels in size -->
                            <Style x:Key="AppBarToggleButtonStyle"
                                   TargetType="AppBarToggleButton"
                                   BasedOn="{StaticResource AppBarToggleButtonRevealStyle}">
                                <Setter Property="Width"
                                        Value="{ThemeResource MTCMediaButtonWidth}" />
                                <Setter Property="Height"
                                        Value="{ThemeResource MTCMediaButtonHeight}" />
                                <Setter Property="AllowFocusOnInteraction"
                                        Value="True" />
                            </Style>
                            <!-- New CommandBar Style -->
                            <Style x:Key="CommandBarStyle"
                                   TargetType="CommandBar">
                                <Setter Property="Height"
                                        Value="{ThemeResource MTCMediaButtonHeight}" />
                                <Setter Property="Background"
                                        Value="Transparent" />
                            </Style>
                            <!-- Style for Error Message text -->
                            <Style x:Key="MediaTextBlockStyle"
                                   TargetType="TextBlock">
                                <Setter Property="VerticalAlignment"
                                        Value="Center" />
                                <Setter Property="Foreground"
                                        Value="{ThemeResource SystemControlForegroundBaseHighBrush}" />
                                <Setter Property="FontSize"
                                        Value="{ThemeResource MTCMediaFontSize}" />
                                <Setter Property="FontFamily"
                                        Value="{ThemeResource MTCMediaFontFamily}" />
                                <Setter Property="Style"
                                        Value="{ThemeResource CaptionTextBlockStyle }" />
                                <Setter Property="IsTextScaleFactorEnabled"
                                        Value="False" />
                            </Style>
                            <!-- Style for Position slider used in Media Transport Controls -->
                            <Style x:Key="MediaSliderStyle"
                                   TargetType="Slider">
                                <Setter Property="Background"
                                        Value="{ThemeResource SliderTrackFill}" />
                                <Setter Property="BorderThickness"
                                        Value="{ThemeResource SliderBorderThemeThickness}" />
                                <Setter Property="Foreground"
                                        Value="{ThemeResource SliderTrackValueFill}" />
                                <Setter Property="FontFamily"
                                        Value="{ThemeResource ContentControlThemeFontFamily}" />
                                <Setter Property="FontSize"
                                        Value="{ThemeResource ControlContentThemeFontSize}" />
                                <Setter Property="ManipulationMode"
                                        Value="None" />
                                <Setter Property="UseSystemFocusVisuals"
                                        Value="{StaticResource UseSystemFocusVisuals}" />
                                <Setter Property="FocusVisualMargin"
                                        Value="-7,0,-7,0" />
                                <Setter Property="IsFocusEngagementEnabled"
                                        Value="True" />
                                <Setter Property="Template">
                                    <Setter.Value>
                                        <ControlTemplate TargetType="Slider">
                                            <Grid Margin="{TemplateBinding Padding}">
                                                <Grid.Resources>
                                                    <Style TargetType="Thumb"
                                                           x:Key="SliderThumbStyle">
                                                        <Setter Property="BorderThickness"
                                                                Value="0" />
                                                        <Setter Property="Background"
                                                                Value="{ThemeResource SliderThumbBackground}" />
                                                        <Setter Property="Foreground"
                                                                Value="{ThemeResource SystemControlBackgroundChromeMediumBrush}" />
                                                        <Setter Property="Template">
                                                            <Setter.Value>
                                                                <ControlTemplate TargetType="Thumb">
                                                                    <Ellipse x:Name="ellipse"
                                                                             Stroke="{TemplateBinding Background}"
                                                                             StrokeThickness="2"
                                                                             Fill="{TemplateBinding Foreground}" />
                                                                </ControlTemplate>
                                                            </Setter.Value>
                                                        </Setter>
                                                    </Style>
                                                    <Style TargetType="ProgressBar"
                                                           x:Key="MediaSliderProgressBarStyle">
                                                        <Setter Property="Height"
                                                                Value="{ThemeResource SliderTrackThemeHeight}" />
                                                        <Setter Property="Minimum"
                                                                Value="0" />
                                                        <Setter Property="Maximum"
                                                                Value="100" />
                                                        <Setter Property="Foreground"
                                                                Value="{ThemeResource SystemControlHighlightChromeAltLowBrush}" />
                                                        <Setter Property="Background"
                                                                Value="Transparent" />
                                                        <Setter Property="BorderBrush"
                                                                Value="Transparent" />
                                                        <Setter Property="BorderThickness"
                                                                Value="1" />
                                                    </Style>
                                                </Grid.Resources>
                                                <Grid.RowDefinitions>
                                                    <RowDefinition Height="Auto" />
                                                    <RowDefinition Height="\*" />
                                                </Grid.RowDefinitions>

                                                <VisualStateManager.VisualStateGroups>
                                                    <VisualStateGroup x:Name="CommonStates">
                                                        <VisualState x:Name="Normal" />

                                                        <VisualState x:Name="Pressed">
                                                            <Storyboard>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalTrackRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackFillPressed}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalTrackRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackFillPressed}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalThumb"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderThumbBackgroundPressed}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalThumb"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderThumbBackgroundPressed}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="SliderContainer"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderContainerBackgroundPressed}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalDecreaseRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackValueFillPressed}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalDecreaseRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackValueFillPressed}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                            </Storyboard>
                                                        </VisualState>

                                                        <VisualState x:Name="Disabled">
                                                            <Storyboard>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HeaderContentPresenter"
                                                                                               Storyboard.TargetProperty="Foreground">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderHeaderForegroundDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalDecreaseRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackValueFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalTrackRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalDecreaseRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackValueFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalTrackRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalThumb"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderThumbBackgroundDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalThumb"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderThumbBackgroundDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="TopTickBar"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTickBarFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="BottomTickBar"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTickBarFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="LeftTickBar"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTickBarFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="RightTickBar"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTickBarFillDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="SliderContainer"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderContainerBackgroundDisabled}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                            </Storyboard>
                                                        </VisualState>

                                                        <VisualState x:Name="PointerOver">
                                                            <Storyboard>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalTrackRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackFillPointerOver}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalTrackRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackFillPointerOver}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalThumb"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderThumbBackgroundPointerOver}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalThumb"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderThumbBackgroundPointerOver}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="SliderContainer"
                                                                                               Storyboard.TargetProperty="Background">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderContainerBackgroundPointerOver}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalDecreaseRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackValueFillPointerOver}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalDecreaseRect"
                                                                                               Storyboard.TargetProperty="Fill">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="{ThemeResource SliderTrackValueFillPointerOver}" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                            </Storyboard>
                                                        </VisualState>
                                                    </VisualStateGroup>
                                                    <VisualStateGroup x:Name="FocusEngagementStates">
                                                        <VisualState x:Name="FocusDisengaged" />
                                                        <VisualState x:Name="FocusEngagedHorizontal">
                                                            <Storyboard>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="SliderContainer"
                                                                                               Storyboard.TargetProperty="(Control.IsTemplateFocusTarget)">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="False" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="HorizontalThumb"
                                                                                               Storyboard.TargetProperty="(Control.IsTemplateFocusTarget)">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="True" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                            </Storyboard>
                                                        </VisualState>
                                                        <VisualState x:Name="FocusEngagedVertical">
                                                            <Storyboard>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="SliderContainer"
                                                                                               Storyboard.TargetProperty="(Control.IsTemplateFocusTarget)">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="False" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                                <ObjectAnimationUsingKeyFrames Storyboard.TargetName="VerticalThumb"
                                                                                               Storyboard.TargetProperty="(Control.IsTemplateFocusTarget)">
                                                                    <DiscreteObjectKeyFrame KeyTime="0"
                                                                                            Value="True" />
                                                                </ObjectAnimationUsingKeyFrames>
                                                            </Storyboard>
                                                        </VisualState>
                                                    </VisualStateGroup>
                                                </VisualStateManager.VisualStateGroups>
                                                <ContentPresenter x:Name="HeaderContentPresenter"
                                                                  x:DeferLoadStrategy="Lazy"
                                                                  Visibility="Collapsed"
                                                                  Foreground="{ThemeResource SliderHeaderForeground}"
                                                                  Margin="{ThemeResource SliderHeaderThemeMargin}"
                                                                  Content="{TemplateBinding Header}"
                                                                  ContentTemplate="{TemplateBinding HeaderTemplate}"
                                                                  FontWeight="{ThemeResource SliderHeaderThemeFontWeight}"
                                                                  TextWrapping="Wrap" />
                                                <Grid x:Name="SliderContainer"
                                                      Background="{ThemeResource SliderContainerBackground}"
                                                      Grid.Row="1"
                                                      Control.IsTemplateFocusTarget="True">
                                                    <Grid x:Name="HorizontalTemplate"
                                                          MinHeight="44">
                                                        <Grid.ColumnDefinitions>
                                                            <ColumnDefinition Width="Auto" />
                                                            <ColumnDefinition Width="Auto" />
                                                            <ColumnDefinition Width="\*" />
                                                        </Grid.ColumnDefinitions>
                                                        <Grid.RowDefinitions>
                                                            <RowDefinition Height="18" />
                                                            <RowDefinition Height="Auto" />
                                                            <RowDefinition Height="18" />
                                                        </Grid.RowDefinitions>
                                                        <Rectangle x:Name="HorizontalTrackRect"
                                                                   Fill="{TemplateBinding Background}"
                                                                   Height="{ThemeResource SliderTrackThemeHeight}"
                                                                   Grid.Row="1"
                                                                   Grid.ColumnSpan="3" />
                                                        <ProgressBar x:Name="DownloadProgressIndicator"
                                                                     Style="{StaticResource MediaSliderProgressBarStyle}"
                                                                     Grid.Row="1"
                                                                     Grid.ColumnSpan="3"
                                                                     HorizontalAlignment="Stretch"
                                                                     VerticalAlignment="Center" />
                                                        <Rectangle x:Name="HorizontalDecreaseRect"
                                                                   Fill="{TemplateBinding Foreground}"
                                                                   Grid.Row="1" />
                                                        <TickBar x:Name="TopTickBar"
                                                                 Visibility="Collapsed"
                                                                 Fill="{ThemeResource SliderTickBarFill}"
                                                                 Height="{ThemeResource SliderOutsideTickBarThemeHeight}"
                                                                 VerticalAlignment="Bottom"
                                                                 Margin="0,0,0,4"
                                                                 Grid.ColumnSpan="3" />
                                                        <TickBar x:Name="HorizontalInlineTickBar"
                                                                 Visibility="Collapsed"
                                                                 Fill="{ThemeResource SliderInlineTickBarFill}"
                                                                 Height="{ThemeResource SliderTrackThemeHeight}"
                                                                 Grid.Row="1"
                                                                 Grid.ColumnSpan="3" />
                                                        <TickBar x:Name="BottomTickBar"
                                                                 Visibility="Collapsed"
                                                                 Fill="{ThemeResource SliderTickBarFill}"
                                                                 Height="{ThemeResource SliderOutsideTickBarThemeHeight}"
                                                                 VerticalAlignment="Top"
                                                                 Margin="0,4,0,0"
                                                                 Grid.Row="2"
                                                                 Grid.ColumnSpan="3" />
                                                        <Thumb x:Name="HorizontalThumb"
                                                               Style="{StaticResource SliderThumbStyle}"
                                                               Height="24"
                                                               Width="24"
                                                               Grid.Row="0"
                                                               Grid.RowSpan="3"
                                                               Grid.Column="1"
                                                               FocusVisualMargin="-14,-6,-14,-6"
                                                               AutomationProperties.AccessibilityView="Raw">
                                                            <ToolTipService.ToolTip>
                                                                <ToolTip x:Name="ThumbnailTooltip">
                                                                    <ContentPresenter Content="{Binding}" />
                                                                </ToolTip>
                                                            </ToolTipService.ToolTip>
                                                            <Thumb.DataContext>
                                                                <Grid Height="112"
                                                                      Width="192">
                                                                    <Image x:Name="ThumbnailImage" />
                                                                    <Border Background="{ThemeResource SystemControlBackgroundBaseMediumBrush}"
                                                                            VerticalAlignment="Bottom"
                                                                            HorizontalAlignment="Left">
                                                                        <TextBlock x:Name="TimeElapsedPreview"
                                                                                   Margin="6,1,6,3"
                                                                                   Style="{StaticResource BodyTextBlockStyle}"
                                                                                   IsTextScaleFactorEnabled="False"
                                                                                   Foreground="{ThemeResource SystemControlPageTextBaseMediumBrush}" />
                                                                    </Border>
                                                                </Grid>
                                                            </Thumb.DataContext>
                                                        </Thumb>
                                                    </Grid>
                                                    <Grid x:Name="VerticalTemplate"
                                                          MinWidth="44"
                                                          Visibility="Collapsed">
                                                        <Grid.RowDefinitions>
                                                            <RowDefinition Height="\*" />
                                                            <RowDefinition Height="Auto" />
                                                            <RowDefinition Height="Auto" />
                                                        </Grid.RowDefinitions>
                                                        <Grid.ColumnDefinitions>
                                                            <ColumnDefinition Width="18" />
                                                            <ColumnDefinition Width="Auto" />
                                                            <ColumnDefinition Width="18" />
                                                        </Grid.ColumnDefinitions>
                                                        <Rectangle x:Name="VerticalTrackRect"
                                                                   Fill="{TemplateBinding Background}"
                                                                   Width="{ThemeResource SliderTrackThemeHeight}"
                                                                   Grid.Column="1"
                                                                   Grid.RowSpan="3" />
                                                        <Rectangle x:Name="VerticalDecreaseRect"
                                                                   Fill="{TemplateBinding Foreground}"
                                                                   Grid.Column="1"
                                                                   Grid.Row="2" />
                                                        <TickBar x:Name="LeftTickBar"
                                                                 Visibility="Collapsed"
                                                                 Fill="{ThemeResource SliderTickBarFill}"
                                                                 Width="{ThemeResource SliderOutsideTickBarThemeHeight}"
                                                                 HorizontalAlignment="Right"
                                                                 Margin="0,0,4,0"
                                                                 Grid.RowSpan="3" />
                                                        <TickBar x:Name="VerticalInlineTickBar"
                                                                 Visibility="Collapsed"
                                                                 Fill="{ThemeResource SliderInlineTickBarFill}"
                                                                 Width="{ThemeResource SliderTrackThemeHeight}"
                                                                 Grid.Column="1"
                                                                 Grid.RowSpan="3" />
                                                        <TickBar x:Name="RightTickBar"
                                                                 Visibility="Collapsed"
                                                                 Fill="{ThemeResource SliderTickBarFill}"
                                                                 Width="{ThemeResource SliderOutsideTickBarThemeHeight}"
                                                                 HorizontalAlignment="Left"
                                                                 Margin="4,0,0,0"
                                                                 Grid.Column="2"
                                                                 Grid.RowSpan="3" />
                                                        <Thumb x:Name="VerticalThumb"
                                                               Style="{StaticResource SliderThumbStyle}"
                                                               DataContext="{TemplateBinding Value}"
                                                               Width="24"
                                                               Height="8"
                                                               Grid.Row="1"
                                                               Grid.Column="0"
                                                               Grid.ColumnSpan="3"
                                                               FocusVisualMargin="-6,-14,-6,-14"
                                                               AutomationProperties.AccessibilityView="Raw" />
                                                    </Grid>
                                                </Grid>
                                            </Grid>
                                        </ControlTemplate>
                                    </Setter.Value>
                                </Setter>
                            </Style>
                            <!-- Style for Volume Flyout used in Media Transport Controls -->
                            <Style x:Key="FlyoutStyle"
                                   TargetType="FlyoutPresenter">
                                <Setter Property="Background"
                                        Value="{ThemeResource MediaTransportControlsFlyoutBackground}" />
                                <Setter Property="Padding"
                                        Value="0" />
                            </Style>
                        </Grid.Resources>

                        <VisualStateManager.VisualStateGroups>
                            <VisualStateGroup x:Name="TitlePanelVisibilityStates">
                                <VisualState x:Name="TitlePanelFadeIn">

                                    <Storyboard>
                                        <DoubleAnimationUsingKeyFrames Storyboard.TargetProperty="Opacity"
                                                                       Storyboard.TargetName="TitlePanelVisibilityStates\_Border">
                                            <EasingDoubleKeyFrame KeyTime="0"
                                                                  Value="0" />
                                            <EasingDoubleKeyFrame KeyTime="0:0:0.3"
                                                                  Value="1" />
                                        </DoubleAnimationUsingKeyFrames>
                                        <DoubleAnimation Storyboard.TargetProperty="Y"
                                                         Storyboard.TargetName="TranslateVerticalTop"
                                                         From="-50"
                                                         To="0.5"
                                                         Duration="0:0:0.3" />
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="TitlePanelFadeOut">

                                    <Storyboard>
                                        <DoubleAnimationUsingKeyFrames Storyboard.TargetProperty="Opacity"
                                                                       Storyboard.TargetName="TitlePanelVisibilityStates\_Border">
                                            <EasingDoubleKeyFrame KeyTime="0"
                                                                  Value="1" />
                                            <EasingDoubleKeyFrame KeyTime="0:0:0.7"
                                                                  Value="0" />
                                        </DoubleAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="IsHitTestVisible"
                                                                       Storyboard.TargetName="TitlePanelVisibilityStates\_Border">
                                            <DiscreteObjectKeyFrame KeyTime="0"
                                                                    Value="False" />
                                        </ObjectAnimationUsingKeyFrames>
                                        <DoubleAnimation Storyboard.TargetProperty="Y"
                                                         Storyboard.TargetName="TranslateVerticalTop"
                                                         From="0.5"
                                                         To="-50"
                                                         Duration="0:0:0.7" />
                                    </Storyboard>
                                </VisualState>

                            </VisualStateGroup>
                            <!-- ControlPanel Visibility states -->
                            <VisualStateGroup x:Name="ControlPanelVisibilityStates">
                                <VisualState x:Name="ControlPanelFadeIn">

                                    <Storyboard>
                                        <DoubleAnimationUsingKeyFrames Storyboard.TargetProperty="Opacity"
                                                                       Storyboard.TargetName="ControlPanel\_ControlPanelVisibilityStates\_Border">
                                            <EasingDoubleKeyFrame KeyTime="0"
                                                                  Value="0" />
                                            <EasingDoubleKeyFrame KeyTime="0:0:0.3"
                                                                  Value="1" />
                                        </DoubleAnimationUsingKeyFrames>
                                        <DoubleAnimation Storyboard.TargetProperty="Y"
                                                         Storyboard.TargetName="TranslateVertical"
                                                         From="50"
                                                         To="0.5"
                                                         Duration="0:0:0.3" />
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="ControlPanelFadeOut">

                                    <Storyboard>
                                        <DoubleAnimationUsingKeyFrames Storyboard.TargetProperty="Opacity"
                                                                       Storyboard.TargetName="ControlPanel\_ControlPanelVisibilityStates\_Border">
                                            <EasingDoubleKeyFrame KeyTime="0"
                                                                  Value="1" />
                                            <EasingDoubleKeyFrame KeyTime="0:0:0.7"
                                                                  Value="0" />
                                        </DoubleAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="IsHitTestVisible"
                                                                       Storyboard.TargetName="ControlPanel\_ControlPanelVisibilityStates\_Border">
                                            <DiscreteObjectKeyFrame KeyTime="0"
                                                                    Value="False" />
                                        </ObjectAnimationUsingKeyFrames>
                                        <DoubleAnimation Storyboard.TargetProperty="Y"
                                                         Storyboard.TargetName="TranslateVertical"
                                                         From="0.5"
                                                         To="50"
                                                         Duration="0:0:0.7" />
                                    </Storyboard>
                                </VisualState>

                            </VisualStateGroup>
                            <!-- ControlPanel Visibility states -->
                            <!-- Media state visual states -->
                            <VisualStateGroup x:Name="MediaStates">
                                <VisualState x:Name="Normal" />
                                <VisualState x:Name="Buffering">
                                    <VisualState.Setters>
                                        <Setter Target="BufferingProgressBar.Visibility"
                                                Value="Visible" />
                                    </VisualState.Setters>
                                </VisualState>
                                <VisualState x:Name="Loading">
                                    <VisualState.Setters>
                                        <Setter Target="BufferingProgressBar.Visibility"
                                                Value="Visible" />
                                    </VisualState.Setters>

                                    <Storyboard>
                                        <DoubleAnimation Storyboard.TargetName="ProgressSlider"
                                                         Storyboard.TargetProperty="Opacity"
                                                         To="0"
                                                         Duration="0" />
                                        <DoubleAnimation Storyboard.TargetName="MediaControlsCommandBar"
                                                         Storyboard.TargetProperty="Opacity"
                                                         To="0"
                                                         Duration="0" />
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Error">
                                    <VisualState.Setters>
                                        <Setter Target="ErrorBorder.Visibility"
                                                Value="Visible" />
                                    </VisualState.Setters>
                                </VisualState>

                                <VisualState x:Name="Disabled">
                                    <Storyboard />
                                </VisualState>

                            </VisualStateGroup>
                            <!-- Media state visual states -->
                            <!-- Audio Selection Button visibility states -->
                            <VisualStateGroup x:Name="AudioSelectionAvailablityStates">
                                <VisualState x:Name="AudioSelectionAvailable">
                                    <VisualState.Setters>
                                        <Setter Target="AudioTracksSelectionButton.Visibility"
                                                Value="Visible" />
                                    </VisualState.Setters>
                                </VisualState>
                                <VisualState x:Name="AudioSelectionUnavailable" />

                            </VisualStateGroup>
                            <!-- Video volume visibility states -->
                            <!-- Closed Captioning Selection Button visibility states -->
                            <VisualStateGroup x:Name="CCSelectionAvailablityStates">
                                <VisualState x:Name="CCSelectionAvailable">
                                    <VisualState.Setters>
                                        <Setter Target="CCSelectionButton.Visibility"
                                                Value="Visible" />
                                    </VisualState.Setters>
                                </VisualState>
                                <VisualState x:Name="CCSelectionUnavailable" />

                            </VisualStateGroup>
                            <!-- Closed Captioning  visibility states -->
                            <!-- Focus states -->
                            <VisualStateGroup x:Name="FocusStates">
                                <VisualState x:Name="Focused">

                                    <Storyboard>
                                        <DoubleAnimation Storyboard.TargetName="FocusVisualWhite"
                                                         Storyboard.TargetProperty="Opacity"
                                                         To="1"
                                                         Duration="0" />
                                        <DoubleAnimation Storyboard.TargetName="FocusVisualBlack"
                                                         Storyboard.TargetProperty="Opacity"
                                                         To="1"
                                                         Duration="0" />
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Unfocused" />
                                <VisualState x:Name="PointerFocused" />

                            </VisualStateGroup>
                            <!-- Focus states -->
                            <VisualStateGroup x:Name="MediaTransportControlMode">
                                <VisualState x:Name="NormalMode" />


                                <VisualState x:Name="CompactMode">
                                    <VisualState.Setters>
                                        <Setter Target="LeftSidePlayBorder.Visibility"
                                                Value="Collapsed" />
                                        <Setter Target="TimeTextGrid.Visibility"
                                                Value="Collapsed" />
                                        <Setter Target="MediaTransportControls\_Command\_Border.(Grid.Column)"
                                                Value="2" />
                                        <Setter Target="MediaTransportControls\_Command\_Border.(Grid.Row)"
                                                Value="1" />
                                        <Setter Target="MediaControlsCommandBar.Margin"
                                                Value="0" />
                                        <!--<Setter Target="PlayPauseButton.Visibility"
                                                Value="Collapsed" />-->
                                    </VisualState.Setters>
                                </VisualState>

                            </VisualStateGroup>
                            <!-- PlayPause states -->
                            <VisualStateGroup x:Name="PlayPauseStates">
                                <VisualState x:Name="PlayState" />


                                <VisualState x:Name="PauseState">
                                    <VisualState.Setters>
                                        <Setter Target="PlayPauseSymbolLeft.Symbol"
                                                Value="Pause" />
                                        <Setter Target="PlayPauseSymbol.Symbol"
                                                Value="Pause" />
                                    </VisualState.Setters>
                                </VisualState>

                            </VisualStateGroup>
                            <!-- VolumeMute states -->
                            <VisualStateGroup x:Name="VolumeMuteStates">
                                <VisualState x:Name="VolumeState" />
                                <VisualState x:Name="MuteState">
                                    <VisualState.Setters>
                                        <Setter Target="AudioMuteSymbol.Symbol"
                                                Value="Mute" />
                                        <Setter Target="VolumeMuteSymbol.Symbol"
                                                Value="Mute" />
                                    </VisualState.Setters>
                                </VisualState>

                            </VisualStateGroup>
                            <!-- FullWindow states -->
                            <VisualStateGroup x:Name="FullWindowStates">
                                <VisualState x:Name="NonFullWindowState">

                                </VisualState>
                                <VisualState x:Name="FullWindowState">
                                    <VisualState.Setters>
                                        <Setter Target="FullWindowSymbol.Symbol"
                                                Value="BackToWindow" />
                                    </VisualState.Setters>
                                </VisualState>

                            </VisualStateGroup>
                            <!-- Repeat states -->
                            <VisualStateGroup x:Name="RepeatStates">
                                <VisualState x:Name="RepeatNoneState" />
                                <VisualState x:Name="RepeatOneState">
                                    <VisualState.Setters>
                                        <Setter Target="RepeatSymbol.Symbol"
                                                Value="RepeatOne" />
                                        <Setter Target="RepeatButton.IsChecked"
                                                Value="True" />
                                    </VisualState.Setters>
                                </VisualState>
                                <VisualState x:Name="RepeatAllState">
                                    <VisualState.Setters>
                                        <Setter Target="RepeatButton.IsChecked"
                                                Value="True" />
                                    </VisualState.Setters>
                                </VisualState>

                            </VisualStateGroup>

                        </VisualStateManager.VisualStateGroups>
                        <!--顶部panel-->
                        <Border x:Name="TitlePanelVisibilityStates\_Border">
                            <Grid x:Name="TitlePanelGrid"
                                  Background="Transparent"
                                  VerticalAlignment="Top"
                                  RenderTransformOrigin="0.5,0.5">
                                <Grid.RenderTransform>
                                    <TranslateTransform x:Name="TranslateVerticalTop" />
                                </Grid.RenderTransform>
                                <TextBlock x:Name="TitleTextBlock"
                                           FontSize="16"
                                           Margin="0,12,0,0"
                                           HorizontalAlignment="Center"
                                           Text="{Binding Path=VideoTitle,Mode=TwoWay,RelativeSource={RelativeSource TemplatedParent}}" />
                            </Grid>
                        </Border>
                        <Border x:Name="ControlPanel\_ControlPanelVisibilityStates\_Border">
                            <Grid x:Name="ControlPanelGrid"
                                  Background="Transparent"
                                  VerticalAlignment="Bottom"
                                  RenderTransformOrigin="0.5,0.5">
                                <Grid.RenderTransform>
                                    <TranslateTransform x:Name="TranslateVertical" />
                                </Grid.RenderTransform>

                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="Auto" />
                                    <ColumnDefinition Width="\*" />
                                    <ColumnDefinition Width="Auto" />
                                </Grid.ColumnDefinitions>

                                <Grid.RowDefinitions>
                                    <RowDefinition Height="Auto" />
                                    <RowDefinition Height="\*" />
                                    <RowDefinition Height="0" />
                                </Grid.RowDefinitions>
                                <Border x:Name="ErrorBorder"
                                        Width="320"
                                        Height="96"
                                        Grid.ColumnSpan="3"
                                        HorizontalAlignment="Center"
                                        Background="{ThemeResource MediaTransportControlsPanelBackground}"
                                        Visibility="Collapsed">
                                    <TextBlock x:Name="ErrorTextBlock"
                                               Style="{StaticResource MediaTextBlockStyle}"
                                               TextWrapping="WrapWholeWords"
                                               Margin="12" />
                                </Border>
                                <Border x:Name="MediaTransportControls\_Timeline\_Border"
                                        Grid.Column="1"
                                        Grid.Row="1">
                                    <Grid x:Name="MediaTransportControls\_Timeline\_Grid">
                                        <Grid.ColumnDefinitions>
                                            <ColumnDefinition Width="Auto" />
                                            <ColumnDefinition Width="\*" />
                                            <ColumnDefinition Width="Auto" />
                                        </Grid.ColumnDefinitions>
                                        <Slider x:Name="ProgressSlider"
                                                Style="{StaticResource MediaSliderStyle}"
                                                Margin="12,0,0,14"
                                                MinWidth="80"
                                                Height="33"
                                                Grid.Column="1"
                                                VerticalAlignment="Center"
                                                IsThumbToolTipEnabled="False" />
                                        <ProgressBar x:Name="BufferingProgressBar"
                                                     Height="4"
                                                     IsIndeterminate="True"
                                                     IsHitTestVisible="False"
                                                     VerticalAlignment="Top"
                                                     Margin="0,2,0,0"
                                                     Visibility="Collapsed" />
                                        <Grid x:Name="TimeTextGrid"
                                              Margin="8,0,0,8"
                                              Grid.Row="1"
                                              Grid.Column="0"
                                              HorizontalAlignment="Left"
                                              VerticalAlignment="Center">
                                            <!--<TextBlock x:Name="TimeElapsedElement"
                                                       Style="{StaticResource MediaTextBlockStyle}"
                                                       Margin="0"
                                                       Text="00:00"
                                                       />-->

                                        </Grid>
                                        <Grid x:Name="TimeStartTextGrid"
                                              Margin="4,0,0,0"
                                              Grid.Row="1"
                                              Grid.Column="0"
                                              HorizontalAlignment="Left"
                                              VerticalAlignment="Center">
                                            <StackPanel Orientation="Horizontal">
                                                <AppBarButton x:Name="PlayPauseButtonOnLeft"
                                                              Margin="0,0,0,4"
                                                              Height="40"
                                                              Style="{StaticResource AppBarButtonStyle}">
                                                    <AppBarButton.Icon>
                                                        <SymbolIcon x:Name="PlayPauseSymbolLeft"
                                                                    Symbol="Play" />
                                                    </AppBarButton.Icon>
                                                </AppBarButton>
                                                <TextBlock x:Name="TimeElapsedElement"
                                                           Style="{StaticResource MediaTextBlockStyle}"
                                                           Text="00:00"
                                                           Margin="0,0,0,10" />
                                            </StackPanel>


                                        </Grid>
                                        <Grid x:Name="TimeReminTextGrid"
                                              Margin="8,0,0,0"
                                              Grid.Row="1"
                                              Grid.Column="2"
                                              HorizontalAlignment="Right"
                                              VerticalAlignment="Center">
                                            <TextBlock x:Name="TimeRemainingElement"
                                                       Style="{StaticResource MediaTextBlockStyle}"
                                                       Text="00:00"
                                                       Margin="0,0,0,10" />
                                        </Grid>
                                    </Grid>
                                </Border>
                                <Border x:Name="LeftSidePlayBorder"
                                        Grid.Column="0"
                                        Grid.Row="1"
                                        Visibility="Visible">

                                    <!--<AppBarButton x:Name="PlayPauseButtonOnLeft"
                                                  Margin="0"
                                                  Height="40"
                                                  Style="{StaticResource AppBarButtonStyle}">
                                        <AppBarButton.Icon>
                                            <SymbolIcon x:Name="PlayPauseSymbolLeft"
                                                        Symbol="Play" />
                                        </AppBarButton.Icon>
                                    </AppBarButton>-->
                                </Border>
                                <AppBarButton x:Name='CastButton'
                                              Style='{StaticResource AppBarButtonStyle}'
                                              MediaTransportControlsHelper.DropoutOrder='11'
                                              Visibility="Collapsed"
                                              Grid.Row="2">
                                    <AppBarButton.Icon>
                                        <FontIcon Glyph="" />
                                    </AppBarButton.Icon>
                                </AppBarButton>
                                <Border x:Name="MediaTransportControls\_Command\_Border"
                                        Grid.Column="2"
                                        Grid.Row="1">
                                    <CommandBar x:Name="MediaControlsCommandBar"
                                                Margin="0,0"
                                                Style="{StaticResource CommandBarStyle}"
                                                IsDynamicOverflowEnabled="False">
                                        <CommandBar.PrimaryCommands>
                                            <AppBarButton x:Name="PlayPauseButton"
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='23'
                                                          Visibility="{Binding ElementName=StopButton,Path=Visibility}">
                                                <AppBarButton.Icon>
                                                    <SymbolIcon x:Name="PlayPauseSymbol"
                                                                Symbol="Play" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <!--清晰度切换-->
                                            <AppBarButton x:Name='PlayInfoButton'
                                                          Margin="8,0,0,0"
                                                          Style="{StaticResource AppBarButtonStyle}"
                                                          Content="{Binding ElementName=PlayInfoListView,Path=SelectedItem.name}">
                                                <AppBarButton.Flyout>
                                                    <Flyout x:Name="PlayInfoFlyout"
                                                            ShowMode="TransientWithDismissOnPointerMoveAway">
                                                        <ListView x:Name="PlayInfoListView"
                                                                  Width="52"
                                                                  ItemsSource="{Binding Path=PlayInfos,Mode=TwoWay,RelativeSource={RelativeSource TemplatedParent}}"
                                                                  SelectedIndex="{Binding Path=PlayInfo\_SelectedIndex,Mode=TwoWay,RelativeSource={RelativeSource TemplatedParent}}">
                                                            <ListView.ItemTemplate>
                                                                <DataTemplate>
                                                                    <TextBlock Text="{Binding name}" />
                                                                </DataTemplate>
                                                            </ListView.ItemTemplate>
                                                        </ListView>
                                                    </Flyout>
                                                </AppBarButton.Flyout>
                                            </AppBarButton>
                                            <AppBarButton x:Name='VolumeMuteButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='19'>
                                                <AppBarButton.Flyout>
                                                    <Flyout x:Name="VolumeFlyout"
                                                            FlyoutPresenterStyle="{StaticResource FlyoutStyle}">
                                                        <StackPanel Orientation="Horizontal">
                                                            <AppBarButton x:Name='AudioMuteButton'
                                                                          Style='{StaticResource AppBarButtonStyle}'
                                                                          VerticalAlignment='Center'
                                                                          HorizontalAlignment='Center'
                                                                          Margin='12'>
                                                                <AppBarButton.Icon>
                                                                    <SymbolIcon x:Name="AudioMuteSymbol"
                                                                                Symbol="Volume" />
                                                                </AppBarButton.Icon>
                                                            </AppBarButton>
                                                            <Slider x:Name='VolumeSlider'
                                                                    Value='50'
                                                                    IsThumbToolTipEnabled='False'
                                                                    Width='{ThemeResource MTCHorizontalVolumeSliderWidth}'
                                                                    VerticalAlignment='Center'
                                                                    HorizontalAlignment='Center'
                                                                    Margin='0' />
                                                            <TextBlock x:Name='VolumeValue'
                                                                       Style='{StaticResource MediaTextBlockStyle}'
                                                                       Text="{Binding ElementName=VolumeSlider,Path=Value}"
                                                                       HorizontalAlignment='Center'
                                                                       VerticalAlignment='Center'
                                                                       Width='24'
                                                                       Margin='12' />
                                                        </StackPanel>
                                                    </Flyout>
                                                </AppBarButton.Flyout>
                                                <AppBarButton.Icon>
                                                    <SymbolIcon x:Name="VolumeMuteSymbol"
                                                                Symbol="Volume" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <AppBarButton x:Name='CCSelectionButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='15'
                                                          Visibility='Collapsed'>
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <AppBarButton x:Name='AudioTracksSelectionButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='13'
                                                          Visibility='Collapsed'>
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <!--<AppBarSeparator x:Name='LeftSeparator'
                                                             Height='0'
                                                             Width='0'
                                                             Margin='0,0' />-->
                                            <AppBarButton x:Name='StopButton'
                                                          Icon='Stop'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='7'
                                                          Visibility='Collapsed' />
                                            <AppBarButton x:Name='SkipBackwardButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='5'
                                                          Visibility='Collapsed'>
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <AppBarButton x:Name='PreviousTrackButton'
                                                          Icon='Previous'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='3'
                                                          Visibility='Collapsed' />
                                            <AppBarButton x:Name='RewindButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='1'
                                                          Visibility='Collapsed'>
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>

                                            <AppBarButton x:Name='FastForwardButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='1'
                                                          Visibility='Collapsed'>
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <AppBarButton x:Name='NextTrackButton'
                                                          Icon='Next'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='3'
                                                          Visibility='Collapsed' />
                                            <AppBarButton x:Name='SkipForwardButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='5'
                                                          Visibility='Collapsed'>
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <AppBarButton x:Name='PlaybackRateButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='10'
                                                          Visibility='Collapsed'>
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <!--<AppBarSeparator x:Name='RightSeparator'
                                                             Height='0'
                                                             Width='0'
                                                             Margin='0,0' />-->
                                            <AppBarToggleButton x:Name='RepeatButton'
                                                                Style='{StaticResource AppBarToggleButtonStyle}'
                                                                MediaTransportControlsHelper.DropoutOrder='1'
                                                                Visibility='Collapsed'>
                                                <AppBarToggleButton.Icon>
                                                    <SymbolIcon x:Name="RepeatSymbol"
                                                                Symbol="RepeatAll" />
                                                </AppBarToggleButton.Icon>
                                            </AppBarToggleButton>
                                            <AppBarButton x:Name='ZoomButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='9'
                                                          Visibility="Collapsed">
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <!--<AppBarButton x:Name='CastButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='11'
                                                          Visibility="Collapsed">
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>-->
                                            <AppBarButton x:Name='CompactOverlayButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='24'
                                                          Visibility="Visible">
                                                <AppBarButton.Icon>
                                                    <FontIcon Glyph="" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                            <AppBarButton x:Name='FullWindowButton'
                                                          Style='{StaticResource AppBarButtonStyle}'
                                                          MediaTransportControlsHelper.DropoutOrder='17'
                                                          Visibility="Visible">
                                                <AppBarButton.Icon>
                                                    <SymbolIcon x:Name="FullWindowSymbol"
                                                                Symbol="FullScreen" />
                                                </AppBarButton.Icon>
                                            </AppBarButton>
                                        </CommandBar.PrimaryCommands>
                                    </CommandBar>
                                </Border>
                            </Grid>
                        </Border>

                    </Grid>

                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</ResourceDictionary>
```