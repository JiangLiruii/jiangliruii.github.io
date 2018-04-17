---
layout:     post
title:      "HTTPS"
subtitle:   "强迫症的我如何实现博客的小绿标" 
date:       2018-04-16 12:39:00
author:     "Lorry"
header-img: "http://p799phkik.bkt.clouddn.com/lock.jpeg"
catalog: true
multilingual: false
tags:
    - 科技(tech)
---
# 起因
### 虽然我的博客里没有任何地方会请求到访者输入任何敏感信息,但是每次看到网址旁边有一个感叹号就不爽  
### 百度了一下(原谅对于fq不稳定的痛心疾首),大部分都采用的[CloudFlare](https://www.cloudflare.com/).
> A Growing Global Network Built for Scale  

### 没想到还能有CDN的功效,不过鉴于博客的访问量还不大,暂时就不考虑付费了.
### 建立安全连接的教程都差不多.我简单归纳下吧,以免前来参观的你还是不够明白.
- 注册一个账号
- 很重要的一步**一定要将servername修改为cliudflare指定地址**,这需要在你自己的网站域名提供商那里进行修改,修改完成之后到目前为止是无法进行http地址映射的,因为DNS的建立需要一定的时间,需要将你的信息不断的传递到全球各地的DNS,在完成之前,DNS都还暂时是之前的,所以,此时是无法立即完成HTTPS安全链接的修改的.下面能做什么呢?
- 等至多72个小时
- 当**statue**变成**active**的时候,就可以进行下一步操作了.
- 进入Crypto,修改SSL为**Flexible SSL**
- 之后就可以对**page rule**进行重定向修改,setting 选择always HTTPS
    - 主域名 http://website.com
    - 所有子域名http://website.com/*
- 再等10分钟左右.

* 先暂时写到这儿,明天再来补图.

## 今日感想:
> 凡事都需要一个仪式感,只有拥有了仪式感才能让这件事情充满意义.