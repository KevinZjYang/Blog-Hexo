---
title: CentOS MongoDb定时备份
tags:
  - CentOS
  - MongoDB
categories:
  - - 开发笔记
date: 2020-03-19 02:36:25
---

本文参照[这篇文章](https://www.cnblogs.com/mengfangui/p/12162337.html)，并修改了部分内容，使其能按预想的运行。

## **1、简述**

通过centos 脚步来执行备份操作，使用crontab实现定时功能，并删除指定天数前的备份。

## **2、创建Mongodb数据库备份目录**
```
mkdir -p /home/backup/mongod\_bak/mongod\_bak\_now
mkdir -p /home/backup/mongod\_bak/mongod\_bak\_list
```
## **3、新建Mongodb数据库备份脚本**

`vi /home/crontab/mongod.sh` #新建文件，输入以下代码

```
#!/bin/bash
MONGODB\_DIR=/usr/local/mongodb/bin  #mongodb安装的bin目录地址
OUT\_DIR=/home/backup/mongod\_bak/mongod\_bak\_now #临时备份目录
TAR\_DIR=/home/backup/mongod\_bak/mongod\_bak\_list #备份存放路径
DATE=\`date +%Y\_%m\_%d\` #获取当前系统时间
DAYS=7 #DAYS=7代表删除7天前的备份，即只保留最近7天的备份
TAR\_BAK="mongod\_bak\_$DATE.tar.gz" #最终保存的数据库备份文件名
cd $OUT\_DIR
rm -rf $OUT\_DIR/\*
mkdir -p $OUT\_DIR/$DATE
cd $MONGODB\_DIR 
#下一行的“-d 数据库 -u 用户名 -p 密码”，请更换为自己数据库的信息
./mongodump -h 127.0.0.1:2017 -d 数据库 -u 用户名 -p 密码 -o $OUT\_DIR/$DATE #备份数据库
tar -zcvf $TAR\_DIR/$TAR\_BAK $OUT\_DIR/$DATE #压缩为.tar.gz格式
find $TAR\_DIR/ -mtime +$DAYS -delete #删除7天前的备份文件
```

**注意**：  
1、`MONGODB_DIR=/usr/local/mongodb/bin` 一定要修改为你安装MongoDb的bin目录。  
2、上面的`./mongodump -h 127.0.0.1:27017 -d 数据库 -u 用户名 -p 密码 -o $OUT_DIR/$DATE` 一定要替换成自己的信息。 `127.0.0.1:27017`数据库端口默认为27017，如果你修改过，请修改为正确的值。

## **4、修改文件属性，使其可执行**

`chmod +x /home/crontab/mongod.sh`

## **5、添加定时运行任务**

`vim /etc/crontab`

最下面输入：

`1  1  \*  \*  \* root   cd /home/crontab && ./mongod.sh`

**6、重新启动crond使设置生效**
```
/sbin/service crond reload #重新载入配置
chkconfig --level 35 crond on  #加入开机自动启动:
/sbin/service crond start   #启动服务
```
每天在/home/backup/mongod\_bak/mongod\_bak\_list目录下面可以看到mongod\_bak\_xxxx\_xx\_xx.tar.gz这样的压缩文件。