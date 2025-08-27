---
title: CozyHosting-HTB靶机
published: 2024-01-27
description: 'HackThebox-Easy难度靶机，从信息收集至root权限全过程'
image: 'image.png'
tags: ['Linux','HTB','Easy']
category: '靶机'
draft: false 
lang: ''
---
* 因原服务器到期，该文章从原服务器迁移至此，图片可能模糊，见谅
# 信息收集
nmap扫描开放端口，目标靶机开放80，22端口。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B2B17DA0E-232C-4F22-9931-BFE905298828%7D.png)
老规矩，访问80端口，跳转地址写到hosts文件中
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BB2DAE37C-C540-4E64-9024-4608C967D02A%7D.png)
访问靶机网址，浏览系统功能点，存在一些展示页和登录功能。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BE5416029-060A-445D-8E62-6BACD018A3F1%7D.png)
看了看login功能，是一个前端框架搭建的登录界面，那就不找框架漏洞了，上dirsearch扫描目录。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BB6590960-F4CC-44CA-BCFE-02F70F4585C3%7D.png)
从搜索结果看，很明显是springboot框架。访问/actuator接口，看到开放了env接口，但是env接口中敏感信息均显示******，而且没有开放能够配合解密的接口。那就关注一下sessions和mappings接口，mappings接口泄露一些路径信息，如/admin、/addhosts、/webjars等。sessions接口中泄露的kanserson用户的session信息，替换原有session后访问/admin，成功进入后台。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B9F8957EB-0EDE-47D8-92AA-35E1735A9D4D%7D.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827134935.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/image-28.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/image-25.png)
寻找后台功能点，存在一个ssh远程连接功能
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827135152.png)
测试一下，尝试连接127.0.0.1提示key无效，尝试连接kali提示host未添加。因为在mappings接口中看到了addhost接口，这里把大部分时间拿去寻找这个接口，测来测去总是提示500错误，随机另找出路。后续注意到ssh远程连接功能是向executessh接口发送post请求，既然是执行ssh，是否可以进行rce，随即展开测试。在username参数进行命令拼接，通过${IFS}绕过空格限制，成功执行命令。（要注意cookie时效性，过期后重新替换cookie）
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BACCC405C-F53B-4FFA-BCF1-62CA14D2167E%7D.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B7BEB66EF-3043-415B-BA8F-33F9C9578999%7D.png)
尝试反弹shell，直接只用命令没法成功反弹，多次尝试最终通过base64编码执行反弹。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BE5D1731B-C7F7-4C99-8C17-98A66C01CEAA%7D.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BA612260B-8CCF-4B43-9A23-B5B42CD18F6B%7D.png)
# ssh立足点
切换交互式shell
``` shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
ctrl + z
stty raw -echo; fg
``` 
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B467C1A79-6B57-4A72-9F93-19BAAFD98942%7D.png)
信息收集，发现/app目录下有一个jar包，通常jar包中会有很多敏感信息，python起一个http服务，下载到kali。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827135414.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827135432.png)
下载速度太慢，直接在靶机中分析。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B07CE0DB4-5386-4EC3-A441-BCE03DA26948%7D.png)
获取到数据库用户+口令，尝试连接。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BD589F9C4-6BE7-4D60-84BD-40FE5640C774%7D.png)
\d命令查看有那些表，select * from users; 拿到用户名和哈希。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827135535.png)
先关注admin用户，尝试破解hash，cmd5查询无果，然后用字典慢慢跑。

john识别一下hash类型，之后用hashcat跑。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827135559.png)
看到hash类型Blowfish，搜索hashcat中对应的编号，为3200
拿kali中rockyou字典跑
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827135630.png)
马上有了结果。

拿到密码后尝试ssh登录。看一下passwd文件，发现存在root、app、postgres、josh能用shell，成功ssh登录josh账户。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BDE969F0D-2464-4CA0-A3A9-4007C1BD3C0D%7D.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BF1BC793C-9AF7-47D1-9AA8-544C1A762290%7D.png)
拿到第一个flag
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827135722.png)
# 提权至ROOT
未拿到ssh用户之前想过提升权限，尝试内核提权发现没有make命令，大多数exp没法编译。

到这拿到ssh用户后尝试suid提权，find / -perm -u=s -type f 2>/dev/null
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B8770192F-D4D9-4B02-9616-0A1072C3188F%7D.png)
能以root执行sudo，sudo又能执行ssh，GTFOBins中查一下ssh怎么提权
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B0A1503BE-31B6-4B68-982B-E7A9563326BB%7D.png)
命令一把索，拿到root用户flag。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B212E9EA4-FF8D-4BC3-B0D0-BCB2DD817C67%7D.png)