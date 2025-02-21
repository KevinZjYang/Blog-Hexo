---
title: UWP设置文件的默认保存位置
tags:
  - UWP
categories:
  - - 开发笔记
date: 2020-03-13 22:22:09
---

原先以为UWP只能使用默认的那几个库。但是后来发现别人的应用中可以随意设置默认的文件保存位置。于是找到了[林德熙大佬的这篇文章](https://blog.csdn.net/lindexi_gd/article/details/52724417/)。

重点是`Windows.Storage.AccessCache`这个命名空间中的`FutureAccessList`类可以保存或者获取最近使用的文件或文件夹。不过获取文件或者文件夹时需要用的一个Token，可以在添加文件或文件夹的时候保存下来。

## 1、保存文件夹

        /// <summary>
        /// 设置默认的下载文件夹
        /// </summary>
        /// <returns>
        /// </returns>
        internal static async Task SetDefaultPathAsync()
        {
            FolderPicker picker = new FolderPicker();
            picker.FileTypeFilter.Add(".png");
            var folder = await picker.PickSingleFolderAsync();
            if (folder != null)
            {
                var token = StorageApplicationPermissions.FutureAccessList.Add(folder);
               // 把token保存到了本地设置 
AppSettingsHelper.SaveLocalSetting(AppLocalSettings.ImageSavePathTokenKey, token);
            }
            else
            {
                // NotifyPopup是自己定义的通知
                var notify = new NotifyPopup("选择文件夹失败，请重试");
                notify.Show();
            }
        }

## 2、获取文件夹

这里在未获取到token，或者获取文件夹结果为`null`时，在图片库新建一个和应用名称相同的文件夹。

        /// <summary>
        /// 获取默认的下载文件夹
        /// </summary>
        /// <returns>
        /// </returns>
        internal async static Task<StorageFolder> GetImageFolderAsync()
        {
           // 从本地设置取出token
            var token = AppSettingsHelper.GetLocalSetting(AppLocalSettings.ImageSavePathTokenKey);
            if (token == null)
            {
                return await KnownFolders.PicturesLibrary.CreateFolderAsync("AppDisplayName".GetLocalized(), CreationCollisionOption.OpenIfExists);
            }
            else
            {
                var folder = await StorageApplicationPermissions.FutureAccessList.GetFolderAsync(token);
                if (folder == null)
                {
                    return await KnownFolders.PicturesLibrary.CreateFolderAsync("AppDisplayName".GetLocalized(), CreationCollisionOption.OpenIfExists);
                }
                else
                {
                    return folder;
                }
            }
        }

PS:还可以使用系统自带的照片app来设置默认下载路径。以前发现的彩蛋。

\[post id=1187\]