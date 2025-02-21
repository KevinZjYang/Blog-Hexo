---
title: U盘安装Exsi6.7 并安装群晖6.2.3-DS918+
tags: 
  - NAS
  - EXSI
categories:
  - - 服务器
date: 2020-12-19 20:24:24
---

## 需要做到的功能

*   使用U盘安装Exsi，因为要直通板载Sata口，没办法。
*   直通板载Sata接口
*   直通显卡，Emby可以识别硬解
*   硬盘顺序正常

## 一、U盘安装EXSI，并直通板载Sata接口

EXSI的安装网上教程很多，此处省略。

Exsi6.7密钥：0U0QJ-FR1EP-KZQN9-J1C74-23P5R

### 1、创建U盘分区来安装群晖

打开Exsi的SSH功能，使用putty，进入Exsi。下面的以我的U盘为例（酷豆）

进入/vmfs/devices/disks目录

```
cd /vmfs/devices/disks
```

输入`ls`获取U盘的名称

```
t10.SanDisk00Cruzer_Fit0000000000004C531000630613103551
```
获取U盘分区`partedUtil getptbl xxx`,其中xxx就是上面获取的U盘名称
```
partedUtil getptbl t10.SanDisk00Cruzer_Fit0000000000004C531000630613103551
```
运行上面命令后得到如下结果：
```
gpt
1919 255 63 30842880
1 64 8191 C12A7328F81F11D2BA4B00A0C93EC93B systemPartition 128
5 8224 520191 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0
6 520224 1032191 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0
7 1032224 1257471 9D27538040AD11DBBF97000C2911D1B8 vmkDiagnostic 0
8 1257504 1843199 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0
9 1843200 7086079 9D27538040AD11DBBF97000C2911D1B8 vmkDiagnostic 0
```

使用第二行的最后1个数字30842880-34得到30842846，替换下面命令最后一行的第三个数字。最后一行的第一个数子设置为和上面几个不一样就行。

```
partedUtil setptbl t10.SanDisk00Cruzer_Fit0000000000004C531000630613103551 gpt \
"1 64 8191 C12A7328F81F11D2BA4B00A0C93EC93B 128" \
"5 8224 520191 EBD0A0A2B9E5443387C068B6B72699C7 0" \
"6 520224 1032191 EBD0A0A2B9E5443387C068B6B72699C7 0" \
"7 1032224 1257471 9D27538040AD11DBBF97000C2911D1B8 0" \
"8 1257504 1843199 EBD0A0A2B9E5443387C068B6B72699C7 0" \
"9 1843200 7086079 9D27538040AD11DBBF97000C2911D1B8 0" \
"2 15472640 30842846 AA31E02A400F11DB9590000C2911D1B8 0"
```

如果上一步失败，输入下面命令后再次输入上面命令

```
esxcli system coredump partition set --enable false
```

创建分区完成，新建数据存储

```
vmkfstools -C vmfs5 -b 1m -S UsbDatastore t10.SanDisk00Cruzer_Fit0000000000004C531000630613103551:2
```

输入`reboot`重启后，就可以在Exsi看到名为UsbDatastore的存储空间了。

### 2、直通板载Sata口

做着一步记得主板sata口要连接硬盘。否则可能识别不到。

1、SSH连接Exsi执行如下命令

```
lspci -v |grep "Class 0106" -B 1
```

查看是否有如下显示

```
0000:00:1f.2 SATAcontroller Mass storage controller: Intel Corporation Lynx Point AHCIController [vmhba0]

         Class 0106: 8086:8c02
```

2、手工配置直通。

```
vi /etc/vmware/passthru.map
```

添加如下：

```
#Intel Corporation Lynx Point AHCI Controller
8086  8c02  d3d0     false
```

保存，重启即可。

## 二、安装群晖

### 1、制作Exsi引导，打上核显驱动

PS:系统上装有VMware才可以使用这种方法.

1）使用StarWindConverter，把下载的Ds918+的1.04b引导，转换为exsi可用的虚拟文件。教程多的很自己找。转换后得到两个文件。

```
synoboot.vmdk
synoboot-flat.vmdk
```

2）右键第一个文件，选择`映射虚拟磁盘...`，或弹出一个对话框，然后选择第二个分区，取消只读模式，点确定，就可以在Windows资源管理器看到这个磁盘了。

![](https://sn3302files.storage.live.com/y4m4AAidumQiTgM7XJZc1SoKEXJGsk_7w8rBllOlNd3vxQlidTYuco1SIAyk6NptXlUZIpUcQeeyq36O4sVq2wkddHiq-czBHIGi-Bj1aFdPGQMDJZubKWXFuYQ4l4_qsIt3V1clH0MP35Eg9uUzJmAY3vIVWTnow-SXk4Zj10-8RQgdsQo_J1jK8p6ROUWfe9b?width=541&height=398&cropmode=none)

3）进入磁盘，删除.lzma后缀的两个文件，用我们下载的带补丁的文件替换。

![](https://sn3302files.storage.live.com/y4mf4X0DY-Bl-XtfyAoHNJRMqC3oHtVH2g8dT1knoIMaQp3JWp47qsStWio3QvDd3VXt9wl9TyNdZAzYqzOuPyOox0yyAg1Kr0Sr1ipq8zNwj2NLRKGQ1zXekhUDFDP8lRb7CKjNzPLFuSbp_dqUUWO9GYtExUmNhgEI1NTlcWcZ9KbngmN7f5DDrKE3-zVIF3D?width=748&height=226&cropmode=none)

4）回到引导所在的文件夹，还是右键第一个文件，点击`断开虚拟磁盘连接`，这样引导文件核显补丁就修改好了。

### 2、安装群晖

把上一步制作好的文件，上传到EXSI。安装群晖的其他步骤省略。主要注意以下几点：

*   系统选择Ubuntu x64
*   磁盘模式 选 独立-持久
*   删掉SCSI控制器、USB、光驱
*   网络选E1000e
*   添加Pci设备，显卡和直通的主板Sata控制器
*   启动时选择引导选项的第三个

群晖修改sn、mac详见下文

\[post id=1246\]

其实可以顺便把下面"三、有硬盘不识别或者顺序错乱"这个做了。

### 3、修复6.2.3导致引导损坏无法升级的问题

1、ssh登录群晖  
2、# sudo -i 进入root模式（输密码的时候不显示，直接输入就好，输完回车）  
3、把FixSynoboot.sh使用群晖的File Station传到任意文件夹，然后右键-属性，复制地址，替换下面命令中文件的地址。

```
cp /volume1/folder/FixSynoboot.sh /usr/local/etc/rc.d
```

回到ssh，输入替换路径后的上面地址。  
4、再输入下面的命令

```
chmod 0755 /usr/local/etc/rc.d/FixSynoboot.sh
```

5、重启群晖

## 三、有硬盘不识别或者顺序错乱

和修改sn、mac方式一样，进入`grub.cfg`

找到 `set sata_args='SataPortMap=4'` 改成set `sata_args='DiskIdxMap=0F00'`

重启，就可以。

## 四、查看核显是否可用

1、群晖开始SSH端口，连接到群晖，输入以下命令

```
ls -la /dev/dri
```

执行命令啥后啥都没有，说明没有核显驱动可用。如果出现下图这样的说明成功了。

![](https://sn3302files.storage.live.com/y4mKyzKSrNN9WokVDbOdJiNQ01fxRju5HcxYv3onBlWEXca0WZwKW4YqP0iROV0j1KMU0Ez0Bhqx3N13clqf_22oAREYm8WqPPA-EBuCZoQc4pYe_V0Qnaz8UzDPLz3StuZe4vl5RSlbJ6iRi-S4Nuc6zebH4DBK5RfuwzP_YRMx0CFTERZz6Qz7lwSToHcGK4_?width=661&height=417&cropmode=none)

进入emby，设置-转码，启用硬件加速，选择高级。如下图所示，编码器和解码器有内容说明可以了。

![](https://sn3302files.storage.live.com/y4mU1vmgSBUc3RnHbdrq3D7dzA_nXLdUngLJ9Uz8DT-7oK6ESlvaJwFx5NrecBJ_snut5beUd6UNdtppo7lBLlrNQZFMHUGg3zdAFTXzd0dEgK7mUK3rUwbRsa876ZhTMfLn6Dlj3hXqa_VqVVsYJqkrQa5jTs-X52whyyvIt5D5dbwUySYfZqL9PKCpn253C6M?width=863&height=734&cropmode=none)

相关文件下载