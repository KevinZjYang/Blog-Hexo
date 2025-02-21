---
title: 使用 Nginx 在 Linux 上托管 ASP.NET Core 应用程序
tags:
  - ASP.NET Core
categories:
  - - 开发笔记
date: 2019-08-12 01:24:20
---

> 本文在[这篇文章](https://www.cnblogs.com/esofar/p/8043792.html)上，经过自己的实践，做了补充。

## 安装 .NET Core SDK

[微软官方文档](https://dotnet.microsoft.com/learn/dotnet/hello-world-tutorial/install) 各个系统都有详细的教程。安装完成后输入`dotnet --version`可以常看安装的版本，同时也可以证明有没有安装成功。

## 部署 ASP.NET Core 应用程序

下面就尝试把微软介绍“[_使用 ASP.NET Core 和 MongoDB 创建 Web API_](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-2.2&tabs=visual-studio)” 创建的Demo，部署到安装 .NET SDK 的 CentOS 系统（下文简称服务器）中。  
服务器采用阿里云轻量应用服务器，系统CentOS7.3。  
如何安装MongoDB请移步下面这篇文章。

\[post id=1173\]

然后借助 FTP 工具 WinSCP 把程序文件传输到服务器`/home/publish`文件夹。

上传完毕后，需要先通过`cd`命令进入网站根目录`/home/publish` ，再输入如下命令启动网站程序：

```
dotnet BooksApi.dll 
```

如果你可以看到如下界面则表示程序启动成功。

![](https://ofbvxq.sn.files.1drv.com/y4mivg7jKE3gigpjQrKt8zB7KHVjTEJzv8HPJLMdGYHnFQCETnJhnagmTZVnpylGXyQDBjnOAAHjMm2tDdjp2DTv2_n6vj-EGTjOSE9BTFB5LAucrcKg3YfI8FB5wS5O15Y3w-WqwRaCjh37a0WHrRF27VWRgjLGGgN_RM0ZA4ESKhgUAyDaBHni4n5ak7k1qhnXf4CojJajtdMCOIsoQcA4Q?width=1190&height=242&cropmode=none)

但是这样项目是无法在后台自动运行的，需要设置进程守护。

## Supervisor 配置守护进程

[Supervisor](http://supervisord.org/) 是用 Python 开发的 Linux/Unix 系统下的一个进程管理工具。它可以使进程脱离终端，变为后台守护进程（daemon）。实时监控进程状态，异常退出时能自动重启。

Supervisor 不支持任何版本的 Window 系统；仅支持在 Python2.4 或更高版本，但不能在任何版本的 Python 3 下工作。

其主要组成部分：

**supervisord**：Supervisor 的守护进程服务，用于接收进程管理命令；

**supervisorctl**：Supervisor 命令行工具，用于和守护进程通信，发送管理进程的指令；

**Web Server**：Web 端进程管理工具，提供与 supervisorctl 类似功能，管理进程；

**XML-RPC Interface**：提供 XML-RPC 接口，请参阅 [XML-RPC API文档](http://supervisord.org/api.html#xml-rpc)。

### 安装 Supervisor

联网状态下，官方推荐首选安装方法是使用`easy_install`，它是[setuptools](http://peak.telecommunity.com/DevCenter/setuptools)（Python 包管理工具）的一个功能。所以先执行如下命令安装 setuptools：

```
yum install python-setuptools
```

请更换`root`用户，执行如下命令安装 Supervisor：

```
easy_install supervisor
```

### 配置 Supervisor

运行`supervisord`服务的时候，需要指定 Supervisor 配置文件，如果没有显示指定，默认会从以下目录中加载：

```
$CWD/supervisord.conf  #$CWD表示运行 supervisord 程序的目录
$CWD/etc/supervisord.conf
/etc/supervisord.conf
/etc/supervisor/supervisord.conf (since Supervisor 3.3.0)
../etc/supervisord.conf (Relative to the executable)
../supervisord.conf (Relative to the executable)
```

所以，先通过如下命令创建目录，以便让 Supervisor 成功加载默认配置：

```
mkdir /etc/supervisor
```

加载目录有了，然后通过`echo_supervisord_conf`程序（用来生成初始配置文件）来初始化一个配置文件：

```
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```

打开`supervisord.conf`文件，可以看到`echo_supervisord_conf`已经帮我们初始化好了一个样例配置，我们需要简单修改一下。

尾部找到如下文本片段：

```
;[include]
;files = relative/directory/*.ini
```

改为：

```
[include]
files = conf.d/*.conf
```

![](https://2rhvlq.sn.files.1drv.com/y4moe345cp0AQ_V-7vLFKWv6KuGQ0yOdVKexTI5pStH2ImviRHx36fwWal21oLHMBCYkD5mDIUwZfwPPWB_DMtlt0yI3Gj_mu7pmvIne2u0iMtlfO7XeoUnfrykYmFujJxcjGDQZSA8YOPBuPo5C0yoMMf0trKGAABkloxT-MQOKKiJ1aBu3wLG_A6M_QC6do65201Z8jR8OnxTeuVFzq2Rhw?width=1054&height=713&cropmode=none)

即，把注释去除、设置`/etc/supervisor/conf.d`为 Supervisor 进程配置文件加载目录。

这样，Supervisor 会自动加载该目录下`.conf`后缀的文件作为共同服务配置。Supervisor 管理的每个进程单独写一个配置文件放在该目录下，`supervisord.conf`配置文件中保留公共配置。

创建进程配置加载目录：

```
mkdir /etc/supervisor/conf.d
```

接下来就需要为我们已经部署的 ASP .NET Core 程序的宿主进程创建一个进程配置文件`netcore.conf`，保存并上传到`/etc/supervisor/conf.d`目录。

配置文件`netcore.conf`内容如下：

```
[program:Scorpio.WebApi]                        ;自定义进程名称
command=dotnet Scorpio.WebApi.dll               ;程序启动命令
directory=/home/wwwroot/scorpio                 ;命令执行的目录
autostart=true                                  ;在Supervisord启动时，程序是否启动
autorestart=true                                ;程序退出后自动重启
startretries=5                                  ;启动失败自动重试次数，默认是3
startsecs=1                                     ;自动重启间隔
user=root                                       ;设置启动进程的用户，默认是root
priority=999                                    ;进程启动优先级，默认999，值小的优先启动
stderr_logfile=/var/log/Scorpio.WebApi.err.log  ;标准错误日志
stdout_logfile=/var/log/Scorpio.WebApi.out.log  ;标准输出日志
environment=ASPNETCORE_ENVIRONMENT=Production   ;进程环境变量
stopsignal=INT                                  ;请求停止时用来杀死程序的信号
```

启动 Supervisor 服务，命令如下：

```
supervisord -c /etc/supervisor/supervisord.conf
```

这时，在会发现我们部署的网站程序不在 shell 中通过`dotnet xxx.dll`启动，同样可以访问。

### 设置 Supervisor 开机启动

首先为 Supervisor 新建一个启动服务脚本`supervisor.service`，然后保存并上传至服务器`/usr/lib/systemd/system/`目录。

脚本内容如下：

```
# supervisord service for systemd (CentOS 7.0+)
# by ET-CS (https://github.com/ET-CS)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

设置开启启动：

```
systemctl enable supervisor
```

验证是否成功：

```
systemctl is-enabled supervisor
```

如果输出`enabled`则表示设置成功，也可重启服务器验证。

> 其它 Linux 发行版开机启动脚本 [User-contributed OS init scripts for Supervisor](https://github.com/Supervisor/initscripts)

### Supervisorctl 管理进程

Supervisor 服务启动后，受其管理的进程会在后台运行。可以通过`supervisorctl`客户端管理进程。

输入如下命令进入`supervisorctl`交互终端，按`Ctrl`+`C`键退出：

```
supervisorctl
```

输入`help`查询帮助：

```
supervisor> help

default commands (type help <topic>):
=====================================
add    exit      open  reload  restart   start   tail
avail  fg        pid   remove  shutdown  status  update
clear  maintail  quit  reread  signal    stop    version
```

输入`help ****`查询详细命令，比如输入`help stop`：

```
supervisor> help stop

stop <name>             Stop a process
stop <gname>:*          Stop all processes in a group
stop <name> <name>      Stop multiple processes or groups
stop all                Stop all processes
```

如何启动、停止、重启进程等命令，我这里就不在记录，大家自行查找吧。

除此之外，Supervisor 还提供了 Web 管理界面用来管理进程，如何配置启动请参考官方文档。

## Nginx 配置反向代理

[Nginx](http://nginx.org/) 是一个高性能的Web服务器软件。这是一个比 Apache HTTP Server 更加灵活和轻量级的程序。

我们的网站程序启动的端口是`5000`，可以借助 Nginx 把程序`5000`端口映射到`80`端口。

> [Nginx官方文档](http://nginx.org/en/docs/) & [Nginx开发从入门到精通 - Tengine](http://tengine.taobao.org/book/index.html)

### 安装 Nginx

首先，我们需要在服务器上安装 Nginx。

**Step1：添加 Nginx 存储库**

要添加 CentOS 7 EPEL 仓库，请打开终端并使用以下命令：

```
sudo yum install epel-release
```

**Step2：安装 Nginx**

现在 Nginx 存储库已经安装在您的服务器上，请使用以下`yum`命令安装 Nginx：

```
sudo yum install nginx
```

**Step3：启动 Nginx**

Nginx 不会自行启动。要运行 Nginx，请输入：

```
sudo systemctl start nginx
```

如果您正在运行防火墙，请运行以下命令以允许 HTTP 和 HTTPS 通信：

```
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

此时，可以在本机的浏览器中访问服务器的 IP 地址`http://ip`来验证 Nginx 是否成功运行。ip指你的服务器外网地址。

如果能看到 Nginx 的默认转发网页则说明一切正常。如下截图：

[![Nginx](https://images2018.cnblogs.com/blog/785976/201807/785976-20180714090720761-223878730.png)](https://images2018.cnblogs.com/blog/785976/201807/785976-20180714090720761-223878730.png)

如果拒绝访问，考虑服务器`80`端口是否开放。可尝试通过下面两条命令开放`80`端口、重启防火墙使修改即时生效。

```
firewall-cmd --zone=public --add-port=80/tcp --permanent
systemctl restart firewalld
```

### 设置 Nginx 开机启动

避免开机需要手动开启 Nginx，可以通过如下快捷命令把 Nginx 配置成系统服务，并设置为开机启动：

```
systemctl enable nginx  #设置开机启动
```

其他命令：

```
systemctl disable nginx   #禁止开机启动
systemctl status nginx     #查看运行状态
systemctl restart nginx    #重启服务
```

### 修改 Nginx 配置文件

首先，拿到 Nginx 的默认配置文件`/etc/nginx/nginx.conf`，把默认`80`端口转发配置`server`节点用`#`符注释掉。

![](https://pz0uiw.sn.files.1drv.com/y4mfmI0cJfUdKTdCCXz0FAcR0cV7MAky-JZjzrhqOFyL0Gpz4iJBrsdFxOU4sOhT2iQgZe12eGoLNagsLumJsjCOv6RDaYpQSDxMKqKO-RMykfAGVXIEUJQ907QJ9HPKEM0YNT2EtCaFGSZo-I6YSwQ1prXQEUCRydbTeFsOkPvRUhSvIfkbFzdIUGRSpMG8lGJw6PPvNIymYY9PnaGYOhU7Q?width=931&height=762&cropmode=none)

然后，我们新建一个配置文件`netcore.conf`，内容如下：

```
server {
    listen 80;
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

保存并上传到 Nginx 的配置加载目录`/etc/nginx/conf.d`，最后执行命令`nginx -s reload`重载 Nginx 配置，使其生效。

在本地浏览器上访问服务器地址，运行结果如下：

![](https://pmig6a.sn.files.1drv.com/y4mSf6UFmROJzZjwf1U4WTfZUkn-KdKZpvOw-4pKefVlhz-YyMykFp7AoG_BcWgJgioBRyearzz0vPqTAQyT4ChIaIBK6P6XqoECbg2aLMlEKcaomiypUD82ARDtqbDXJ7hJGXLV2HB53cJZIp1Nnw3SbuILoUx2er1SG8KrCVw-AEWug1MUKkrkbhvxC7if3SXuDjdaoBODqjQQc-gaAF3hw?width=956&height=320&cropmode=none)

这个问题是由于 SELinux 保护机制所导致，我们需要将 Nginx 添加至 SELinux 的白名单。执行命令：

```
yum install policycoreutils-python

sudo cat /var/log/audit/audit.log  grep nginx  grep denied  audit2allow -M mynginx

sudo semodule -i mynginx.pp
```

PS:如果执行命令时出现下列情况：

执行:

```
sudo cat /var/log/audit/audit.log  grep nginx  grep denied  audit2allow -M mynginx
```

返回的是`nothing to do`的话，说明项目还没有跑起来，需要先把项目跑起来，先去上一步设置守护进程。

执行 `sudo cat /var/log/audit/audit.log grep nginx grep denied audit2allow -M mynginx`  
时出现：`you must specify the -p option with the path to the policy file`

请先检查SELinux是否被禁用掉了

在/etc/sysconfig下有一个SeLinux文件，使用记事本打开，更改其中的SELINUX项的值就可以了。

SELINUX=disable  禁用SeLinux  
SELINUX=enforcing  使用SeLinux  
重要：保存完后要重启 reboot

再次访问，运行结果如下：

![](https://bt83ow.sn.files.1drv.com/y4mKsnOp3SzxITngOTkzv7lLxOPvVC-MeZu1JW_SLGVDV6QCwCvz2Krb23jjHnEJ4GXBbJ2BnYUcPGqUP-jpdCEFdtusTpyz2Iu_dvKA8Ph1bJSspxwfIU0rEPxSX-5PgVRGKPdKdVFQwg8j5EOag932mE3rX6dCHoY0YK0nEJf5xvjJ2KHgunOUL81emf_ZaNR2hWGwPvq3UiA0OQoPkYeRw?width=724&height=273&cropmode=none)

可以看到，访问的接口成功返回数据，证明 Nginx 已经完成对我们部署应用程序的转发。

至此，我们已经完成了 ASP.NET Core 应用程序在 CentOS7 服务器上的部署。

> 参考文章：
> 
> [Centos7运行NETCore完整教程（三）：nginx反向代理](https://blog.csdn.net/zt102545/article/details/88545833#3.%E5%B0%86Nginx%E6%B7%BB%E5%8A%A0%E5%88%B0SELinux%E7%99%BD%E5%90%8D%E5%8D%95)
> 
> [使用 Nginx 在 Linux 上托管 ASP.NET Core 应用程序](https://www.cnblogs.com/esofar/p/8043792.html)