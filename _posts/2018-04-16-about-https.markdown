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
### 虽然我的博客里没有任何地方会请求到访者输入任何敏感信息,但是每次看到网址旁边有一个感叹号就不爽, 而且因为是非安全网站,无法使用google的Service Worker来使服务离线化.
### 谷歌解决办法,土豪版的过滤掉之后大部分都采用的[CloudFlare](https://www.cloudflare.com/).
> A Growing Global Network Built for Scale  

### 没想到还能有CDN的功效,不过没有国内的加速点,对于我注册的海外站点倒是无所谓,肯定会有速度上的提升,如果是国内的站点,请使用国内的CDN加速吧,用cf会更慢.亲自ping过,海外站点使用cf后延迟减少了一半..

### 建立安全连接的教程都差不多.我的简单归纳:
- 注册一个账号
- 很重要的一步**一定要将servername修改为cliudflare指定地址**,这需要在你自己的网站域名提供商那里进行修改,修改完成之后到目前为止是无法进行http地址映射的,因为DNS的建立需要一定的时间,需要将你的信息不断的传递到全球各地的DNS,在完成之前,DNS都还暂时是之前的,所以,此时是无法立即完成HTTPS安全链接的修改的.下面能做什么呢?
- 等至多72个小时
- 当**statue**变成**active**的时候,就可以进行下一步操作了.
- 进入Crypto,修改SSL为**Flexible SSL**(如果你有SSL证书可选择Full SSL)
- 之后就可以对**page rule**进行重定向修改,setting 选择always HTTPS, 一定要看到已激活界面![激活界面](http://p799phkik.bkt.clouddn.com/image/POSTS/cfStatue.png)才能设置为always HTTPS![always https](http://p799phkik.bkt.clouddn.com/image/POSTS/https.png)
    - 主域名 http://website.com
    - 所有子域名http://website.com/*
- 再等10分钟左右.就可以看到正常重定向到HTTPS界面中,有小绿锁了,并且Service Worker也可以正常使用了.



## 今日感想:
> 凡事都需要一个仪式感,只有拥有了仪式感才能让这件事情充满意义.