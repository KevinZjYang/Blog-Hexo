---
title: 在 CentOS7 上安装 MongoDB
tags:
  - CentOS
  - MongoDB
categories:
  - - 开发笔记
date: 2019-08-12 01:22:24
---

## 阿里云服务器安装mongodb

在[原文](https://www.cnblogs.com/wanghy898/p/10901092.html)基础上根据自己体验做了一定的修改。

连接阿里云服务器

1.下载mongodb [查看最新版本](https://www.mongodb.com/download-center/community)

```
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.12.tgz
```

2.解压

```
tar zxvf mongodb-linux-x86_64-4.0.12.tgz 
```

  将解压过后的包移动到指定目录，并重命名为mongodb

```
mv mongodb-linux-x86_64-4.0.12/ /usr/local/mongodb
```

3.创建数据文件夹和日志文件等

```
mkdir -p /usr/local/mongodb/data 
touch /usr/local/mongodb/mongod.log
touch /usr/local/mongodb/mongodb.conf
```

4.启动方式

　进入mongo的bin目录下：

```
cd /usr/local/mongodb/bin
```

通过配置文件启动 首先进行配置

```
vim /usr/local/mongodb/mongodb.conf
```

在配置中加入如下代码

 dbpath=/usr/local/mongodb/data
 logpath=/usr/local/mongodb/mongod.log
 logappend = true 
 port = 27017 
 fork = true 
 maxConns=1000
 noauth = true
 bind\_ip = 0.0.0.0
 storageEngine=wiredTiger

```
port=27017 #端口  
dbpath= /usr/mongodb/db #数据库存文件存放目录  
logpath= /usr/mongodb/mongodb.log #日志文件存放路径  
logappend=true #使用追加的方式写日志  
fork=false #不以守护程序的方式启用，即不在后台运行  
maxConns=100 #最大同时连接数  
noauth=true #不启用验证  
journal=true #每次写入会记录一条操作日志（通过journal可以重新构造出写入的数据）。
#即使宕机，启动时wiredtiger会先将数据恢复到最近一次的checkpoint点，然后重放后续的journal日志来恢复。
storageEngine=wiredTiger  #存储引擎有mmapv1、wiretiger、mongorocks
bind_ip = 0.0.0.0  #这样就可外部访问了，例如从win10中去连虚拟机中的MongoDB
```

加入后按ESC，保存退出

```
:wq
```

进入mongo的bin目录下

```
cd /usr/local/mongodb/bin
```

 再执行

```
./mongod --config /usr/local/mongodb/mongodb.conf
```

```
sudo chmod 777 mongodb.conf
```

启动过程如果出现端口占用则使用  ps aux grep mongod 命令查看端口占用情况

 (2) 停止

```
./mongod -shutdown -dbpath=/usr/local/mongodb/data
```

去阿里云控制台，打开27017端口，最后放开端口（远程访问必须开端口，否则不要开）

```
/sbin/iptables -I INPUT -p tcp --dport 27017 -j ACCEPT
```

验证mongoDB是否启动，输入命令`lsof -i :27017`，监测端口已经在使用中，所以说启动已经完成

在浏览器输入：`http://ip:27017` 如果出现下面的文字，说明是能正常访问MongoDB的。ip是你的服务器外网ip地址。

```
It looks like you are trying to access MongoDB over HTTP on the native driver port.
```

## centos下无法使用lsof命令解决方案

在centos下查看端口占用情况输入命令

```
lsof -i:8080(自己的端口号)
```

出现"-bash: lsof: command not found"

解决方案:

```
yum install lsof
```

安装即可

## 设置为系统服务并且设置开机启动

首先添加MongoDB系统服务，命令如下：`vim /etc/rc.d/init.d/mongod`

打开编辑器后，我们将下面的配置粘贴进去，然后保存

```
#!/bin/bash

export MONGO_HOME=/usr/local/mongodb
#chkconfig:2345 20 90
#description:mongod
#processname:mongod
case $1 in
          start) 
              $MONGO_HOME/bin/mongod --config $MONGO_HOME/mongodb.conf
              ;;
          stop)
              $MONGO_HOME/bin/mongod  --shutdown --config $MONGO_HOME/mongodb.conf
              ;;
          status)
              ps -ef  grep mongod
              ;;
          restart)
              $MONGO_HOME/bin/mongod  --shutdown --config $MONGO_HOME/mongodb.conf
              $MONGO_HOME/bin/mongod --config $MONGO_HOME/mongodb.conf
              ;;
          *)
              echo "require startstopstatusrestart"
              ;;
esac
```

保存完成之后，添加脚本执行权限，命令如下： `chmod +x /etc/rc.d/init.d/mongod` 　　

可以使用命令`service mongod stop`关闭MongoDB服务。

将此服务设置为开机启动，命令如下：`chkconfig mongod on`，然后重新启动机器进行测试，发现开机服务应启动并且端口也在使用中。

验证mongoDB是否启动，输入命令`lsof -i :27017`，监测端口已经在使用中，所以说启动已经完成