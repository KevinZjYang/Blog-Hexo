---
title: 开启代理后如何使用Github Desktop
tags: 
  - Github Desktop
categories:
  - - 开发笔记
date: 2020-09-23 19:36:32
---

1.  在有代理设置的电脑上 clone GitHub 仓库时提示：  
    `fatal: unable to access 'https://github.com/fhqfghgdx/Test-github.git/': Failed to connect to github.com port 443: Timed out`
2.  经分析与收集资料，由于工作机上网需要设置代理，但是GitHub Windows 客户端没有找到网络设置的功能，故网搜如何 GitHub 设置代理，解决如下：  
    找到 .gitconfig 配置文件，在文件结尾添加：  
    `[http] proxy = http:_//www.proxy.com:port_`
3.  如我的设置：  
    `[http] proxy = http:_//127.0.0.1:1081_`
4.  设置之后即可正常使用 GitHub clone 或者 commit 代码。  
      
    注：  
     .gitconfig 配置文件位置说明：默认在个人User个人主目录下，是个隐藏文件，如我的 C:\\Users\\xxxxx\\.gitconfig，也可以直接使用 Everything 或者 Listary 在电脑内全局搜索该文件。

![](https://sn3302files.storage.live.com/y4mx97Kyt9tIpUZ5zkW3VQ0suVgdS5VnoyTZxoQLS8llQOnukuTvF79vjiMqZXSIwuRi8z250vBEs2EC1w30JoC8epHu_ScuONQT8Z4UmAyllmdvJLmKqnrpr3fEN7iN-uwpTZjWefU-AXYVLJQ6c1LZJevgdGMzECfXgk4dOg6QwIeoxMWlAEZMGnxx8kjFy4c?width=423&height=289&cropmode=none)