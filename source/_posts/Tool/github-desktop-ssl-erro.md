---
title: Github Desktop无法使用
tags:
  - Git
  - Github Desktop
categories:
  - - 随笔
date: 2019-04-16 19:23:36
---

## 一、常见错误及处理方法

### 1、OpenSSL Erro及解决方法

![](https://fc8gla.dm.files.1drv.com/y4mP4gFGYnQZj4U3_29-5-p_ZR41PU0K8F7CGv4NGx5l3sXVqoSXEU1Fp7jeSOvJFmb4R0NYcA_S7emMroUlCQJ1trK0S7OhVX5HZPlywJv3rtwmk3XBZ_cTYj9NnbLsyLX_8mCIXtfJs4OUQaltErAM08lHA4_oJSmk-buvWZJyOlhjCPwE_j6eelL7r-oe8775PlifyTV4q-jJIfsTP4FPg?width=630&height=236&cropmode=none)

首先是需要安装[Git](https://git-scm.com/)(安装有不懂就一直下一步)，然后发现直接也用不了。需要打开Git CMD,然后执行下面的命令。看评论说执行第一个就可以使用了。但是我两个都执行了。无所谓了。反正是有效的。[原文地址](https://stackoverflow.com/questions/49345357/fatal-unable-to-access-https-github-com-xxx-openssl-ssl-connect-ssl-error)

```
git config --global http.sslBackend "openssl"
git config --global http.sslCAInfo "C:\Program Files\Git\mingw64\ssl\cert.pem"
```

### 2、请求超时Timeout

本地开启梯子后，Git还需要设置代理，才能正常使用。

添加代理，主要是socks5.稳妥起见都设置了吧。

```
git config --global https.proxy http://127.0.0.1:1081

git config --global https.proxy https://127.0.0.1:1081

git config --global http.proxy 'socks5://127.0.0.1:1080' 

git config --global https.proxy 'socks5://127.0.0.1:1080'
```

取消代理

```
git config --global --unset http.proxy

git config --global --unset https.proxy
```

这样Github Desktop应该就可以正常使用了。

## 二、Git常用使用方法

首先进入项目根目录，空白处右键，选择“Git Bash Here".

![](https://geting.bn.files.1drv.com/y4mla4pr4_m-TcZwRvpmwyeG9dD1zP0l5JZ_xTV0wvxAUXVJ8JsERFgMGD4bUmsKJB5Cdv2hqz6CH_y1KgL8OQMx3646KRpL3DJgky3wMYebOrAu_Mz5KK7OIUzqGnBLWBeEDZwouhDfI7P3H7AG6EoKFNQuf9mHXgLPwVEs0CfBtyBiiOyUJ4kq0gNlDGfTpFOcJCAawt-MT90G9vxUnKJzw?width=899&height=585&cropmode=none)

基本配置邮箱和用户名

```
git config --global user.name "KevinZjYang"
git config --global user.email "keivn.zj.yang@outlook.com"
```

### 1、Pull和Fetch

pull是直接和本地合并的。fetch是更安全的方式，[看这里](https://www.jianshu.com/p/4c1d3b741b19)

```
git pull origin master
```

### 2、Push

其他同上

```
git push origin master
```

### 3、查看远程仓库

```
git remote -v
```

### 4、Clone

首先进入你想克隆的本地文件夹。然后再运行下面命令。

```
git clone 远程项目地址
```

### 5、清除未提交的内容

```
git clean -df
git reset --hard
```