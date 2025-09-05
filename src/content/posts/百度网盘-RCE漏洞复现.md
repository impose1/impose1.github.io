---
title: 百度网盘-RCE漏洞复现
published: 2025-09-05
description: '百度网盘远程命令执行漏洞复现'
image: ''
tags: ['漏洞复现','RCE']
category: '漏洞复现'
draft: false 
lang: ''
---
# 漏洞前提
> 1. 知道目标机器用户名
> 2. 百度网盘安装路径为默认路径/知道网盘安装路径(BaiduNetdisk.exe路径)
> 3. 仅针对windows版本，实验复现7.59.5.104，7.60版本复现未成功
# 参考连接
> https://mrxn.net/news/baidupan-windows-client-rce.html
# 过程
## 成功截图
![](https://impos1.oss-cn-beijing.aliyuncs.com/1976418a-72f6-4533-88b6-dfe4be34a364.png)
## 整体过程
> 1. 百度网盘开机自动监听10000端口(https协议)
> 2. 构造恶意参数uk访问
> 3. 调用baidunetdisk.exe执行命令时拼接uk参数值
> 4. 劫持dll
> 5. 任意命令执行

> POC该文章不给出，本文参考连接里给出了POC
> 需要注意的是文章POC需要稍作调整，包括反斜杠、url编码、网盘安装路径等
> 欢迎各位师傅点击左边联系方式探讨

