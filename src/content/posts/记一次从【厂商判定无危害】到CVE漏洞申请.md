---
title: 记一次从【厂商判定无危害】到CVE漏洞申请
published: 2023-08-27
description: '挖洞过程中捡到的CVE漏洞，已发放编号且已经公布'
image: ''
tags: ['CVE','实战']
category: '实战'
draft: false 
lang: ''
---
* 因原服务器到期，该文章从原服务器迁移至此，图片可能模糊，见谅
# 前言
某天在公司闲来无事挖洞模会鱼，找了一个企业src，搜了一会资产发现一个很简陋的网站，直觉告诉我里面肯定有点东西。网站功能类似与ai生成文本，能生成工作总结、个人总结以及文本润色等。这篇文章只是单纯记录一下这次的奇幻过程，水平有限，漏洞危害不大。
# 开始测试
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B110B54B9-2BBE-4797-A226-BE35C242F1CD%7D.png)
因为感觉很简陋，就以为是厂家自行开发的网站（后面发现并不是），先扫一下路径看看有啥。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BF40405AA-187C-44EF-BB63-E21AD8028713%7D.png)
从结果来看肯定要关注一下/config和/openapi.json，果不其然/openapi.json里面暴露大量接口，并且接口存在未授权。敏感信息泄露肯定是有了，再看看有接口具体信息。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BCE1854A9-0D2A-4822-B0CE-71D34B77C161%7D.png)
看到/proxy接口想到了ssrf，构造一个试一下，发现返回了百度页面，大概率有问题，但无法判断是服务器返回的百度还是客户端请求的百度。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827140224.png)
用dnslog试试，发现记录的ip不是自己的这边的，那必然是目标的。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827140245.png)
存在ssrf能有什么危害?可以尝试探测内网，尝试伪协议读文件、写shell等等，但是经过尝试上面的用法均不可行，也可能是我水平问题。无所谓，到饭点了，敏感信息泄露和ssrf交个低危去吃饭了。回来一看平台判定秒过，判定低危赏金50，想着摸会鱼晚上还加个餐美滋滋，结果第二天厂商判定漏洞无危害。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BA99668F1-E0F3-4364-9B0C-C1593F44EDE8%7D.png)
mmp，想着也就我技术菜没打穿你说无危害，这来个大牛给你拿了我看你还有没有危害。越想越来气，再打开站点看看。
# 二次测试
这次打开站点注意到了网站title和页脚版权信息
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827140348.png)
哦吼，到这才发现可能不是厂商自己写的，可能用的别人搭建的，fofa搜一搜验证一下，直接搜ico文件hash，发现1w多条结果，那必然是用的别人的东西。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827140412.png)
上网搜搜有没有历史漏洞先。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827140449.png)
欸嘿，还真有点东西喽，CVE-2023-34239好像是一个文件读取和上文中的ssrf，CVE中说file接口可以读文件，存在遍历问题，当时尝试读取/etc/passwd无果，遍历路径读取也无果，可能被修复了。于是乎再次查看泄露的接口，想着既然/file接口没做权限控制，那/upload会不会存在同样问题，于是乎根据openapi.json中泄露的upload接口信息做测试 ，发现果然没对文件类型和内容做任何限制，上传文件均保存至系统/tmp目录下。这里尝试了上传路径遍历，很遗憾做了限制，只能保存至/tmp目录。
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BCC249063-82A9-48A1-8F21-D59373CB44D6%7D.png)
上传文件类型没有限制，意味着可以上传任意恶意脚本文件，如果能结合文件包含则可能拿到网站权限。不过该网站为python中gradio生成，不知道该如何下一步，想到这感觉这个洞危害也不大，在考虑cve会不会收这么垃圾的洞，但是有转念一想如果相同服务器下存在其他站点，并且其他站点存在文件包含漏洞就可能会与此联动，抱着试一试的心态交一下cve。
# CVE申请
网上很多cve提交教程，如https://blog.csdn.net/qq_44172732/article/details/131614405

cve申请网站：https://cveform.mitre.org/

填写漏洞信息后几分钟会收到cve官方受理回复，8月29日提交，9月2日收到cve编号收录通知。
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827140540.png)
刚申请漏洞公开，不知道需要几天处理完成。（现在已经处理完成）
# 结语
这次利用摸鱼时间小测一把，期间几乎没什么复杂技术含量，完全没想到能申请到cve，这还是我第一次申请cve。幸好厂商没收漏洞，不然我可能拿着50块疯狂星期四去了，就不会有后面cve申请了。测试过程中其实有很多问题，比如gradio是python的一个库，而我在ssrf漏洞中尝试通过php伪协议读文件就必然不会成功，所以当后端语言为python时ssrf如何利用还要继续研究。
