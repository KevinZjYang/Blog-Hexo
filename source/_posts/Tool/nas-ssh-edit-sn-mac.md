---
title: 使用SSH修改群晖SN/MAC[转]
tags: 
  - Nas
categories:
  - - 服务器
date: 2019-09-28 16:28:35
---

开启SSH端口。

在控制面板里面——>终端机和SNMP ，启动SSH功能打勾，并设置端口（建议别用22，改用其他的，比如220或者其他都行）。

![群晖开启端口](https://wp.gxnas.com/wp-content/uploads/2019/02/ba0eda299e5ce73c2a5272696b878a69.png)

挂载synoboot1分区。

第一步：用ssh工具如xshell连接到群晖的地址，用创建群晖的管理用户登陆。

如：admin   密码 123456

第二步：获取root权限。（如果这一步失败，请按照教程开启root权限，6.17及以下版本看这里，6.2及以上版本看这里。）

`sudo -i`

回车后会提示输入密码，即再次输入当前管理账号的密码。

成功后会变成root@Sxxx:~#， 其中root开头，#结尾。

第三步：在/tmp目录下创建一个临时目录，名字随意，如：boot

`mkdir -p /tmp/boot`

第四步：切换到dev目录

`cd /dev`

第五步：将synoboot1 分区挂载到boot

U盘安装的一般是这个

```
mount -t vfat synoboot1 /tmp/boot/
```
如果出现：`mount: special device synoboot1 does not exist`，使用下面代码

虚拟机装的是这个，一般是sda1，如果调整过硬盘顺序，sdx1中x需要根据具体情况确认。可以通过ls命令查看所有文件，参考群晖中看到的硬盘顺序确认。
```
mount -t vfat sda1 /tmp/boot
```

第六步：切换到/tmp/boot/目录

```
cd /tmp/boot/
ls
```
可以看到挂载后有 bzImage  EFI  grub  info.txt 等文件夹或文件（主机或版本不同时，文件夹、文件名有所差别，但肯定有grub文件夹），至此挂载成功。

第七步：切换到grub目录，修改grub.cfg文件
```
cd grub
vi grub.cfg
```
此时，进入了vim查看 grub.cfg文件。

![修改配置文件](https://wp.gxnas.com/wp-content/uploads/2019/02/412e74dc2321a3ba16a3fc85b7a00ab6.png)

按键盘向下、向右等箭头，将光标移动到要修改的地方 （注：我这里是双网卡）

此时还是命令模式，按键盘上的 i 键（小写状态），进入文档编辑模式，此时就可以输入新的SN，MAC1的新值。

修改完成后，按键盘上的Esc键，返回到命令模式，输入:wq （英文状态的字符），保存并退出。如果修改乱了，想不保存并退出，则是输入 :q! 。

此时可以再  `vi grub.cfg` 进去看看是否修改成功。

最后重启主机，

`reboot`

修改成功了。

原文链接： https://wp.gxnas.com/2615.html
