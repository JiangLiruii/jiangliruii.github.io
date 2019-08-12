---
layout:       post
title:        "RPC 和 REST"
subtitle:     "API 的设计"
date:         2019-07-01 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/mvc-mvp-and-mvvm-design-pattern.png'
catalog:      true
multilingual: false
tags:
    - web
---

# 概述

**RPC(Remote Procedure Call)** 和 **REST(Representational State Transfer)** 都可以作为 Web API 设计方法.

## 基于 HTTP 的实现

两者都可以基于 HTTP 来实现.

### RPC

更适合一个数据量较小的场景, 可以节省带宽并且简化服务.可以具体到任何你想要的东西上, 也就是说可以让一个服务只做一件事情.

面向的是远程调用的一个方法

### REST

更适合请求一个资源, 这个资源根本不会考虑它自己的 functionality(数据的函数性), 值是一个包含数据的地方, REST 好处在于一个资源 API 可以从很多台 clients 中调用, 只关心属于这个特定的 domain 下的所有数据.

面向的是一个资源.

### RPC 和 REST 分别长什么样

以下是**基于 Node.js**的实现

- 静态资源

```js
var http = require('http')

var CAT_ID = 1;

var CATS = [
  {id: CAT_ID, name: '英短', age: 1}
]
```

- RPC 的实现

```js
var app = http.createServer((req, res) => {
  if (req.url === 'getCatNameById?id=' + CAT_ID) {
    var cat = CATS.find(c => c.id === CAT_ID);
    res.setHeader('Content-Type', 'application/json; charset=utf-8');
    res.end(JSON.stringify({name: cat.name}));
  }
})
app.listen(1337)
```

注意到了 url 中的 `getCatNameById` 吗? 这就是RPC **臭名昭著**的将函数调用名写到 url 中.会很方便的理解将要返回的东西以及 url 代表的意思. 这个服务不关心自己的所有可用的性质, 所以是很直接的;

从返回值看两者的差异, REST 返回的是整个的 CATS 资源, 具体怎么处理由业务方负责, 而 RPC 直接返回的是被调用方法的返回值, 即 name.

## 两者的优劣势:

1. REST 在业务方需要对某一个资源进行多种方式的获取或使用该资源的字段的情况下会更具有优势, 比如获得了 CATS 的数据之后, 不仅仅包含 name, 也有 id, age 等信息, 可以一并处理而不用重新再发送一条请求 id 或 age 的请求.
2. RPC 属于一种方法的调用, 在业务方看来跟调用本地的函数没什么区别, 只是需要通过网络请求. 在代码层面上会更加的函数式, 不需要关心具体的资源, 只需要去调用针对某个资源的方法并获取调用的结果即可.

RPC 和 REST 本身并没有高低之分, 需要看具体的业务场景, 两者也不是泾渭分明, 也是有重叠的地方. 比如获取数据库中的某个数据, 那么可以如果业务方只需要一个字段, 那么在一系列的查表操作后只返回该字段的数据, 整个过程更像是 RPC, 但是你说那个字段是不是一种资源呢? 其实也可以算. 总之, 更多的是在于业务方调用接口时的使用姿势和思想, 而没有一个很具体可以区分的标准. **很多事都没有标准**, 是吧?


