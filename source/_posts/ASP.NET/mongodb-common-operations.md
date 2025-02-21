---
title: Mongodb常用操作
tags:
  - MongoDB
categories:
  - - 开发笔记
date: 2020-03-03 18:29:08
---

基于在Centos下安装的Mongodb

1.  进入安装目录`cd /usr/local/mongodb/bin`
2.  ./mongo
3.  显示所有数据库 `show dbs`
4.  切换数据库 `use dbname`
5.  显示所有表单 `show tables`
6.  显示数据库中所有集合`show collections`
7.  显示结合中所有项`db.colllectionname.find().pretty()`
8.  ./mongod --dbpath=/usr/local/mongodb/data -- port 12345 修改默认端口

[详细操作文档](https://www.runoob.com/mongodb/mongodb-create-database.html)

PS:数据库被黑客删了是什么情况

![](https://www.yzj0308.com/wp-content/uploads/2020/03/2020030310492185.png)

数据库名字前被添加”HOW\_TO\_RESTORE“

![](https://www.yzj0308.com/wp-content/uploads/2020/03/2020030310493358.png)

打开以后就是索要btc的。。。