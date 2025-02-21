---
title: ASP.NET Core连接带密码的MongoDb
tags:
  - ASP.NET Core
  - MongoDB
categories:
  - - 开发笔记
date: 2020-03-04 00:27:55
---

mongodb密码和传统数据如mysql等有些区别：

mongodb的用户名和密码是基于特定数据库的，而不是基于整个系统的。所有所有数据库db都需要设置密码

## mongodb设置管理用户和密码

1、`show dbs`  
在mongodb新版本里并没有admin数据库，但是并不妨碍第2步操作。

2、`use admin` 进入admin数据库

3、创建管理员账户  
`db.createUser({ user: "useradmin", pwd: "adminpassword", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })`  
mongodb中的用户是基于身份role的，该管理员账户的 role是 userAdminAnyDatabase。 ‘userAdmin’代表用户管理身份，’AnyDatabase’ 代表可以管理任何数据库。

4、验证第3步用户添加是否成功  
`db.auth("useradmin", "adminpassword")` 如果返回1，则表示成功。  
`exit`退出系统  
`db.auth()`方法理解为 用户的验证功能

5、修改配置  
修改mongodb.conf文件中的noauth=false，如下图

![](https://www.yzj0308.com/wp-content/uploads/2020/03/2020030316373741.png)

6、重启mongodb `sudo service mongod restart`

7、进入mongodb,用第3步的 管理员账户登录，用该账户创建其他数据库管理员账号  
`use admin`  
`db.auth("useradmin", "adminpassword")`

8、新建你需要管理的mongodb 数据的账号密码。  
`use yourdatabase`  
`db.createUser({ user: "youruser", pwd: "yourpassword", roles: [{ role: "dbOwner", db:"yourdatabase" }] })`rote:dbOwner 代表数据库所有者角色，拥有最高该数据库最高权限。比如新建索引等

9、新建数据库读写账户  
`use yourdatabase`  
`db.createUser({ user: "youruser2", pwd: "yourpassword2", roles: [{ role: "readWrite",db: "yourdatabase" }] })`该用户用于该数据的读写，只拥有读写权限。

10、现在数据的用户名和密码就建好了。  
可以使用：`mongodb://youruser2:yourpassword2@localhost:27017/yourdatabase`来链接

## ASP.NET Core里appsettings.json的设置

"WallhavenUpdateLogDatabaseSettings": {
    "WallhavenUpdateLogCollectionName": "youcollectio",
    "ConnectionString": "mongodb://youruser2:yourpassword2@localhost:27017/yourdatabase",
    "DatabaseName": "yourdatabase"

参考文章：[mongodb设置密码](https://juejin.im/post/5b0519cf518825426539d05e)