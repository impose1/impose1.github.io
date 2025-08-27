---
title: Previous-HTB靶机
published: 2025-08-27
description: '2025-08推出的新靶机，从信息收集-打点-root权限全过程'
image: '{BA7EC344-E507-4FF4-8F48-A03A1D1ECAE4}.png'
tags: ['HTB','Linux','Middle']
category: '靶机'
draft: false 
lang: ''
---
## 信息收集
### 端口扫描
> nmap的扫描结果如下图所示，只开放了两个端口，没有进一步的扫描详细端口信息
> ![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B116C8FE2-56A0-4F9C-AEE6-3044A3470601%7D%20(1).png)
访问80端口跳转previous.htb，加入hosts文件，访问域名，看到是一个门户网站
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B736EE907-A47C-4B6E-9350-3D359E1E855D%7D.png)
### 初步查看
看一下有什么功能点，门户左下角泄露了一处邮箱，记下来。
``` 
jeremy@previous.htb
``` 
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B5CF3A4F2-EA20-4F98-944F-BE9D370679DB%7D.png)
还有一处登录框，在signin接口，打开看一眼
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B64909849-1F51-4653-AF3E-DE52B0C48BFE%7D.png)
另外JS里发现几个接口，访问全部跳转登陆页面
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B3DCFF8F9-73CD-46AF-AB04-4B74F13478C1%7D.png)
## 打点/漏洞利用
### 尝试工作
- 使用burp中TsojanScan插件漏扫，无发现
- 子域名探测，无发现
- fuzzAPI接口，目录扫描，没什么有价值的信息

到这都没有可利用信息，因为泄露了一个邮箱，剩一个爆破还没打，一般不进行爆破，回头继续信息收集，本想着扩大字典扫描，但是在不存在的uri报错中发现使用的框架-NextAuth，结合登录框，联想到是否有登陆绕过或者其他相关漏洞
![](https://impos1.oss-cn-beijing.aliyuncs.com/5d6ddec9-edaf-4fb3-b57d-9e10f7f8f038.png)
### 聚焦NextAuth
搜索NextAuth近期是否爆出过什么漏洞，果然发现一个cve，登陆认证绕过
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B7D35B81B-D6E7-40AE-90D3-3A26E0EC522D%7D.png)
拿到poc修改header头，成功访问之前不能访问的/docs路径
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BE154EF6A-71CE-48D7-856E-12FB25146EFD%7D.png)
查找有什么功能点，在example中找到一个下载api
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B8F8AFC21-EB46-4381-9230-05C07EFE9872%7D.png)
接下来我的思路是想读一些配置文件，看看是否有可利用信息
chatgpt询问nextjs目录结构和敏感文件，构造一下访问看看
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B538B9C3A-C177-4988-B8D6-34F23842E7CF%7D.png)
拿到一个secret，应该是用于身份鉴别时加密的，可能是加密jwt的，但是已经做了身份绕过，估计没用，先记下来。
package.json，拿到后询问了ai有什么存在漏洞的组件，返回了前面利用的CVE编号，依旧无果
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B4CBB34D5-30E6-44A1-9E9B-704F95628306%7D.png)
根据chatgpt返回的框架结构，查到一些目录结构
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BA02EE89E-3B83-4F51-A2B3-C4FAD2208363%7D.png)
拿到目录结构后，这里卡了很久，在猜测目录结构，各种拼接，最后根据上面图片，结合ai，在下面url中拿到一个密码
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B3A3F35B2-0384-4A8B-8D8A-4374AD127059%7D.png)
登录时因为在passwd中没看到jeremy用户，先使用node用户ssh登陆，密码不对，后面随手试了一下jeremy(上图中泄露加邮箱泄露)，结果ssh登进去了，进来后发现刚才得nextjs项目是docker环境，解释了为什么passwd文件没有jeremy用户
![](https://impos1.oss-cn-beijing.aliyuncs.com/aa2e51e0-0065-44a7-b26a-0fba054802dc.png)
## 权限提升
### SUDO&nbsp;-l
思路是先手动看有什么利用点，找不到的话传linpeas脚本做信息收集。
这里登陆后随手看了sudo&nbsp;-l，发现没有重置env，意味着我们在运行下面指定的terraform语句时可以自定义env
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BB2AFDCA2-83BA-4920-BC76-82C0A01F251E%7D.png)
直接粘贴给chatgpt，返回可利用结果
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B8B7A1EAC-2E9E-47C8-8911-9C4769B36612%7D.png)
根据gpt给的结果，利用一下，写一个复制bash的脚本
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B196EECA2-71AB-4139-B5BD-C421EF0E2074%7D.png)
建立一个目录
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B2334E0BD-5D54-4C04-B6D0-462B3542D6C2%7D.png)
跑一下拿到root
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BF70CF380-39A1-473D-BCC6-D1D8312A5F6E%7D.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/78cc8695-77e7-4264-a292-0ffee573af47.png)
## 总结
> 信息收集->NextAuth框架登陆绕过->任意文件读取->源码硬编码口令泄露->ssh登陆->sudo环境变量提权
> 对我的难点在任意文件读取时，不知道该读哪个文件，最后配合ai完成框架源码读取发现敏感文件