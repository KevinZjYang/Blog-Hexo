---
title: 阿里云虚拟主机无法使用Smtp发送邮件
tags:
  - Smtp
  - 阿里云
  - wordpress
categories:
  - - 服务器
date: 2019-07-11 17:31:57
---

因阿里云虚拟主机禁用了mail()函数，导致wordpress系统无法使用默认mail()函数发送邮件，即便改用SMTP插件也无法成功发送（因为阿里云虚拟主机提供的是fsockopen，而SMTP所使用的是steam\_connect\_client），因此想要实现SMTP成功发送邮件，就要修改wordpress程序源文件class-smtp.php，把wordpres的SMTP发送方式改为fsockopen方式。

1、登录阿里云主机管理控制台，进入：站点信息-高级环境设置-php.ini设置，PHP函数fsockopen设置：启用

2、修改wordpress系统文件，打开 /wp-includes/class-smtp.php，在274-283 行位置，查找以下代码：

$socket\_context = stream\_context\_create($options);
//Suppress errors; connection failures are handled at a higher level
$this->smtp\_conn = @stream\_socket\_client(
     $host . ":" . $port,
     $errno,
     $errstr,
     $timeout,
     STREAM\_CLIENT\_CONNECT,
     $socket\_context
);

替换为以下代码：

$this->smtp\_conn = @fsockopen($host,$port,$errno,$errstr,$timeout);

然后使用 [Easy WP SMTP](http://www.themege.com/downloads/easy-wp-smtp/) 或 WP MAIL SMTP 这类 SMTP 发送邮件的插件，配置好一般就可以了。  
**特别注意：由于这个方法是直接修改了 WordPress 的核心代码，一旦升级了 WordPress 版本，就需要重新进行修改，切记！！！**

[原文链接](https://www.themege.com/aliyun-mail-failure/)