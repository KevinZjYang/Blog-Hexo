---
title: CentOS nginx简单实现文件服务器
tags:
  - CentOS
  - Nginx
categories:
  - - 开发笔记
date: 2020-04-02 13:12:13
---

1、首先，查看HTTP允许访问的端口。

```
semanage port -l  grep http_port_t
```

可以使用上述端口中的一个（我用的9000），也可以添加一个，比如1234

```
semanage port -a -t http_port_t  -p tcp 1234
```

2、在`/etc/nginx/conf.d/`新建一个文件`file_server.conf`，修改内容为下面所示

```
server {
  listen 9000; #端口 
  server_name localhost;  
  charset utf-8; # 避免中文乱码 
  root /home/download/; # 显示的根索引目录，注意这里要改成你自己的，目录要存在
  location / { 
    autoindex on; #开启索引功能  
    autoindex_exact_size off; #关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）   
    autoindex_localtime on; # 显示本机时间而非 GMT 时间  
  }
}
```

然后重启nginx服务，`service nginx restart`

在浏览器输入你的域名或者IP地址加端口号访问。

```
域名:9000 或者 ip:9000
```

3、如果访问出现`403 Forbidden`，修改`/etc/selinux/config`文件

```
SELINUX=disabled
```

重启`reboot`。

参考文章：

[用nginx一分钟实现文件服务器](https://www.jianshu.com/p/d9f886a9666a)

[nginx报错处理](https://blog.csdn.net/RunSnail2018/article/details/81185138)

[四种解决Nginx出现403 forbidden 报错的方法](https://www.linuxprobe.com/nginx-403-forbidden.html)