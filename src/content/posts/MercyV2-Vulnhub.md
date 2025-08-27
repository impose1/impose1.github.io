---
title: MercyV2-Vulnhub靶机
published: 2023-04-22
description: 'Vulnhub靶机，在OSCP推荐靶机名单'
image: ''
tags: ['Vulnhub','Linux','Easy']
category: '靶机'
draft: false 
lang: ''
---
* 因原服务器到期，该文章从原服务器迁移至此，图片可能模糊，见谅
# 信息收集&打点
> 端口扫描，发现开放smb服务、两个http服务。
> ``` 
> nmap -p- 192.168.75.131 -sV -v --min-rate=10000
> ``` 
![](https://impos1.oss-cn-beijing.aliyuncs.com/asdhiuwoalbgiluwyqauhjsbyui.png)
## 80端口

![](https://impos1.oss-cn-beijing.aliyuncs.com/image-1asdihzxuihasd.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/image-2asdasdihiuoqlwhqw.png)
存在/mercy，/nomercy目录,访问看一下功能点
![](https://impos1.oss-cn-beijing.aliyuncs.com/image-3-1024xijabyaxvj1j231172.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B5AED0C9D-BA1B-4C43-B034-CF8FD4AB84FA%7D.png)
/nomercy搭建的时rips0.53服务，searchsploit查一下有没有相关漏洞
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BDF53788F-4DF2-48E6-8426-FC0BE288AE3E%7D.png)
可以看到RIPS应用服务存在本地文件包含漏洞，下载payload
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B7C657ABC-AC21-4DCF-AF3B-19419B3DD9D0%7D.png)
``` poc
rips/windows/code.php?file=xxxxx
``` 
访问一下看看
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BD2D8045E-54B4-490C-8319-CED0F0B99F9A%7D.png)
能本地包含，读了一堆没用的文件后，80端口先到这里，再大体看一下8080端口的web服务。
## 8080端口
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B3603C3A8-373C-4F87-B5E7-AB24F1643C8B%7D.png)
8080端口为tomcat默认页面，并且提示了User定义在/etc/tomcat7/tomcat-users.xml,结合80端口的文件包含漏洞，读tomcat后台用户口令
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B9916DF87-53EE-419B-A7EA-915A1E081F4E%7D.png)
拿到两个账户
``` 
username="thisisasuperduperlonguser" password="heartbreakisinevitable"

username="fluffy" password="freakishfluffybunny"
``` 
其中thisisasuperduperlonguser写了为admin权限，使用该账户登录tomcat后台。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BE52BEBC9-0897-4ACC-8B47-9C0314F8D5E8%7D.png)
tomcat后台可以通过上传war包getshell，生成一个反弹shell的war包并上传部署
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B890AD7B3-AD65-4645-9871-A3470EED0A82%7D.png)
启监听，浏览器访问http://192.168.75.131:8080/nohacker/，拿到tomcat权限shell
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BD6860854-FBFB-4B6D-86E3-79B80F8511FD%7D.png)
# 权限提升
*“拿到tomcat后，尝试过切fluffy用户，可能口令输错了以为登录不了，然后上传linpeas.sh脚本信息搜集，发现存在内核提权漏洞，但是没有gcc命令，该路不通，发现了靶场处在docker环境下。在卡了很长时间后发现8080端口存在robots.txt，提示存在/tryharder/tryharder页面，访问后是一段文本base64编码，其中提示系统存在弱口令‘password’，又去搜集smb服务暴露信息，发现存在qiu用户，可以通过'password‘拿到其共享文件，看了一圈没啥用。翻了/home的各个用户目录，基本没权限并且没有有用文件，没有收获最后重新试了一下su fluffy用户，才发现口令输错了”*
切换shell交互模式

通过上述搜集到的账户口令切到fluffy用户
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BECC9D8F9-159B-4F22-A540-D9E432808FDF%7D.png)
翻用户目录，发现存在timelock文件，打开发现是个shell脚本，并且刚刚执行过，猜测该脚本会定期执行。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B70B72926-153F-4EA9-8AEE-2C842D951B53%7D.png)
该脚本又为777权限，所有者为root，尝试通过修改该文件进行提权
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B2BA702B6-2094-482F-8720-0EC481C2FC8D%7D.png)
文件后追加两行，复制bash并且给777权限
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BADAA2A0E-76C2-4A26-930C-12D44161A873%7D.png)
等待大约三分中后，成功执行
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B7832A9DC-2F5A-47D4-9B54-8FDD7793C1DB%7D.png)