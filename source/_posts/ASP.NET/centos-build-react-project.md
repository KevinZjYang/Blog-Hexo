---
title: CentOS部署React项目
tags:
  - CentOS
  - React
categories:
  - - 开发笔记
date: 2020-04-14 01:58:15
---

前期工作：[安装最新版node.js](https://www.jianshu.com/p/4a9449506924)

1、git clone 项目地址

2、进入项目目录，npm install

3、npm run build

配置nginx

```
server {
      listen 8009; #任何可用端口都可以
      server_name localhost;
      location / {
            root /home/xxx/build; #生成文件的路径
            try_files $uri $uri/ /index.html last;
            index index.html;
            add_header Access-Control-Allow-Origin *;
      }
}
```

安装node和npm

```
curl –silent –location https://rpm.nodesource.com/setup_13.x  sudo bash -
sudo yum install -y nodejs
#卸载旧版本
sudo yum remove -y nodejs npm
```