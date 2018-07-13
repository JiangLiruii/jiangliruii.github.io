---
layout:       post
title:        "Other Interesting Things"
subtitle:     "积小流以成江河"
date:         2018-07-11 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/promise.jpg'
catalog:      true
multilingual: false
tags:
    - Interesting
---

有时候看到一些有趣的东西,又不知道放在哪里,也担心过目即忘,索性开一个文,用于记录点滴,不定期整理成单独文章.

## [DOMContentLoaded vs Load Event](https://www.sitepoint.com/performance-auditing-a-firefox-developer-tools-deep-dive/)

`DOMContentLoaded`在HTML document 完成加载和实例化后(parsed)触发,不包括CSS stylesheet, images 和 frames.

`load`是当HTML Document andAll Associated stylesheets, images and frames are completely loaded.是在DOMContentLoaded之后,包括了其不包括的内容.

## Network Timings
![](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2017/12/1512623728network-timings-1024x188.png)

### Blockd: 网络连接的排队时间
### DNS resolution: 解析服务器的主机名的时间
### Connecting: 打开TCP连接的时间
### TLS setup: TLS(Transport Layer Security)设置的时间(也有可能是SSL)
### Sending: 向服务器发送请求的时间
### Receiving: 从服务器获取请求的时间(如果有缓存的话为读取缓存的时间)
### Waiting: 客户端接到第一个字节数据前所花费的时间,在Chrome DevTool中是**TTFB(Time To First Byte)**

