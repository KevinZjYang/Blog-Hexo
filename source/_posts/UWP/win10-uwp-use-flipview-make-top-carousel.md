---
title: Win10 UWP 使用FlipView制作顶部轮播控件
tags:
  - FlipView
  - UWP
  - Win10
  - Windows 10
categories:
  - - 开发笔记
date: 2019-04-16 03:03:26
---

先在这儿备份一下。  
还必须凑够一定的字数，否则网页就显示不正常。我也是真心醉了。  
还必须凑够一定的字数，否则网页就显示不正常。我也是真心醉了。

```xml

        <RelativePanel Grid.Row="1">
            <Grid x:Name="GridLeft"
                  RelativePanel.AlignLeftWithPanel="True">
                <FlipView x:Name="LeftFv"
                          ItemsSource="{x:Bind Data.itemList,Mode=OneWay}"
                          IsEnabled="False"
                          BorderThickness="0">
                    <FlipView.ItemTemplate>
                        <DataTemplate
                                      x:DataType="models:Itemlist1">
                            <Image Stretch="UniformToFill">
                                <Image.Source>
                                    <BitmapImage UriSource="{x:Bind data.content.data.cover.feed}" />
                                </Image.Source>
                            </Image>
                        </DataTemplate>
                    </FlipView.ItemTemplate>
                </FlipView>
                <Rectangle>
                    <Rectangle.Fill>
                        <LinearGradientBrush StartPoint="0,0.5"
                                             EndPoint="1,0.5">
                            <GradientStop Color="Black"
                                          Offset="0" />
                            <GradientStop Color="#99000000"
                                          Offset="1" />
                        </LinearGradientBrush>
                    </Rectangle.Fill>
                </Rectangle>
            </Grid>
            <Grid x:Name="GridCenter"
                  RelativePanel.RightOf="GridLeft">
                <FlipView x:Name="CenterFv"
                          ItemsSource="{x:Bind Data.itemList,Mode=OneWay}"
                          BorderThickness="0"
                          SelectionChanged="CenterFv_SelectionChanged">
                    <FlipView.ItemTemplate>
                        <DataTemplate x:DataType="models:Itemlist1">
                            <Grid>
                                <Image ImageOpened="Image_ImageOpened"
                                       Tapped="Image_Tapped"
                                       Stretch="Fill"
                                       HorizontalAlignment="Left"
                                       VerticalAlignment="Top">
                                    <Image.Source>
                                        <BitmapImage UriSource="{x:Bind data.content.data.cover.feed}" />
                                    </Image.Source>
                                </Image>
                                <Grid VerticalAlignment="Bottom"
                                      Background="{ThemeResource SystemControlPageBackgroundChromeLowBrush}"
                                      Opacity=".7"
                                      Padding="8">
                                    <TextBlock Text="{x:Bind data.header.title}"
                                               HorizontalAlignment="Center"
                                               FontSize="16"
                                               VerticalAlignment="Center"
                                               TextTrimming="CharacterEllipsis"
                                               FontWeight="Bold"/>
                                </Grid>
                            </Grid>
                         
                        </DataTemplate>
                    </FlipView.ItemTemplate>
                </FlipView>                          
            </Grid>
            <Grid x:Name="GridRight"
                  RelativePanel.RightOf="GridCenter"
                  RelativePanel.AlignRightWithPanel="True">
                <FlipView x:Name="RightFv"
                          ItemsSource="{x:Bind Data.itemList,Mode=OneWay}"
                          IsEnabled="False"
                          BorderThickness="0">
                    <FlipView.ItemTemplate>
                        <DataTemplate 
                                      x:DataType="models:Itemlist1">
                            <Image Stretch="UniformToFill">
                                <Image.Source>
                                    <BitmapImage UriSource="{x:Bind data.content.data.cover.feed}" />
                                </Image.Source>
                            </Image>
                        </DataTemplate>
                    </FlipView.ItemTemplate>
                </FlipView>
                <Rectangle>
                    <Rectangle.Fill>
                        <LinearGradientBrush StartPoint="1,0.5"
                                             EndPoint="0,0.5">
                            <GradientStop Color="Black"
                                          Offset="0" />
                            <GradientStop Color="#99000000"
                                          Offset="1" />
                        </LinearGradientBrush>
                    </Rectangle.Fill>
                </Rectangle>
            </Grid>
        </RelativePanel>
```

```c#
using KaiYanUWP.Views;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices.WindowsRuntime;
using Windows.Foundation;
using Windows.Foundation.Collections;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Controls.Primitives;
using Windows.UI.Xaml.Data;
using Windows.UI.Xaml.Input;
using Windows.UI.Xaml.Media;
using Windows.UI.Xaml.Media.Imaging;
using Windows.UI.Xaml.Navigation;

//https://go.microsoft.com/fwlink/?LinkId=234236 上介绍了“用户控件”项模板

namespace KaiYanUWP.Controls
{
    public sealed partial class SquareCardUserControl : UserControl
    {


        public Core.Models.Data Data
        {
            get { return (Core.Models.Data)GetValue(DataProperty); }
            set { SetValue(DataProperty, value); }
        }

        // Using a DependencyProperty as the backing store for Data1.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty DataProperty =
            DependencyProperty.Register("Data", typeof(Core.Models.Data), typeof(SquareCardUserControl), new PropertyMetadata(null));


        public SquareCardUserControl()
        {
            this.InitializeComponent();
            Loaded += SquareCardUserControl_Loaded;
            this.SizeChanged += SquareCardUserControl_SizeChanged;
        }

        private void SquareCardUserControl_SizeChanged(object sender, SizeChangedEventArgs e)
        {
            Refresh();
        }

        private void SquareCardUserControl_Loaded(object sender, RoutedEventArgs e)
        {
            Refresh();
            CenterFv.SelectedIndex = 0;
            LeftFv.SelectedIndex = LeftFv.Items.Count - 1;
            RightFv.SelectedIndex = CenterFv.SelectedIndex + 1;
            //this.NaviGridView.SelectedIndex = CenterFv.SelectedIndex;
            DispatcherTimer timer = new DispatcherTimer();
            timer.Interval = new System.TimeSpan(0, 0, 3);
            timer.Tick += (s, args) =>
            {
                this.CenterFv.SelectedIndex = this.CenterFv.SelectedIndex < this.CenterFv.Items.Count - 1 ? ++this.CenterFv.SelectedIndex : 0;
            };
            timer.Start();
        }
        private void Refresh()
        {
            //double density = Windows.Graphics.Display.DisplayInformation.GetForCurrentView().RawPixelsPerViewPixel;
            var width = RootGird.ActualWidth;
            //var height = RootGird.ActualHeight/density;
           
            if ( width >= 480)
            {
                CenterFv.Width = 480;
                var height = CenterFv.Width * 0.5625;
                CenterFv.Height = height;
                var leftWidth = (width-480) / 2;

                GridLeft.Width = leftWidth;
                GridLeft.Height = height;

                GridRight.Width = leftWidth;
                GridRight.Height = height;

            }
            else
            {
                CenterFv.Width = width;
                var height = width * 0.5625;
                CenterFv.Height = height;

                GridLeft.Width = 0;
                GridLeft.Height = height;
                GridRight.Height = height;
                GridRight.Width = 0;

            }
        }
        private void Image_Tapped(object sender, TappedRoutedEventArgs e)
        {
            var data = (e.OriginalSource as FrameworkElement).DataContext as Core.Models.Itemlist1;
            ShellPage.Current.NavigateToDetail("KaiYanUWP.Views.PlayVideoPage",data.data.header.id);
        }

        private void CenterFv_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            if (this.CenterFv.SelectedIndex == 0)
            {
                this.LeftFv.SelectedIndex = this.LeftFv.Items.Count - 1;
                this.RightFv.SelectedIndex = 1;
                //this.NaviGridView.SelectedIndex = CenterFv.SelectedIndex;
            }
            else if (this.CenterFv.SelectedIndex == 1)
            {
                this.LeftFv.SelectedIndex = 0;
                this.RightFv.SelectedIndex = this.RightFv.Items.Count - 1;
                //this.NaviGridView.SelectedIndex = CenterFv.SelectedIndex;
            }
            else if (this.CenterFv.SelectedIndex == this.CenterFv.Items.Count - 1)
            {
                this.LeftFv.SelectedIndex = this.LeftFv.Items.Count - 2;
                this.RightFv.SelectedIndex = 0;
                //this.NaviGridView.SelectedIndex = CenterFv.SelectedIndex;
            }
            else if ((this.CenterFv.SelectedIndex < (this.CenterFv.Items.Count - 1)) && this.CenterFv.SelectedIndex > -1)
            {
                this.LeftFv.SelectedIndex = this.CenterFv.SelectedIndex - 1;
                this.RightFv.SelectedIndex = this.CenterFv.SelectedIndex + 1;
                //this.NaviGridView.SelectedIndex = CenterFv.SelectedIndex;
            }
            else
            {
                return;
            }
        }

        private void Image_ImageOpened(object sender, RoutedEventArgs e)
        {

        }
    }
}
```