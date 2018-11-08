---
layout:       post
title:        "json-server自构api"
subtitle:     "授之于鱼不如授之以渔"
date:         2018-06-19 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/back/json.jpg'
catalog:      true
multilingual: false
tags:
    - React
---

在进行前端开发时会需要后端的配合,比如给我们一些api,获取返回类型.如果需要等到后端把接口写好,服务架起来是需要时间的,这段时间里我们就会阻塞,当然可以先采用模拟数据从本地取,但有没有更好的办法来模拟成与线上环境一模一样的呢?

# 上帝说要有光,于是就有了光.

# json-server

## 简介

官方说仅仅需要3分钟,便可搭建一个json服务器.现在开始计时.

- 安装 json-server
    `cnpm intsall -g json-server`

- 创建 db.json 文件
```json
{
  "posts": [
    { "id": 1, "title": "json-server", "author": "Lorry" }
  ],
  "comments": [
    { "id": 1, "body": "some comment", "postId": 1 }
  ],
  "profile": { "name": "Lorry" }
}
```

创建了一篇文章,一个评论,关联到id1的文章中.

- 启动服务
    `json-server --watch db.json`
    
- 查看服务
    - 进入 `http://localhost:3000/posts/1` 可以得到第一篇文章 `{ "id": 1, "title": "json-server", "author": "Lorry" }`

看看时间花了多久?应该3分钟不违反广告法了.

除了查看之外还可以进行数据库的操作,比如修改,删除,添加等. 它依赖于 [lowdb](https://github.com/typicode/lowdb),并且需要注意在请求头中需要指定内容类型: `Content-Type: application/json`

## 进阶

### 路由 Routes

上例中自动创建了很多路由

- Plural routes 复数形式的路由

```
GET    /posts
GET    /posts/1
POST   /posts
PUT    /posts/1
PATCH  /posts/1
DELETE /posts/1
```

- Singular routes 单数形式
```
GET    /profile
POST   /profile
PUT    /profile
PATCH  /profile
```

- Filter 筛选路由 使用 `.` 访问更深层级的属性,比如author.name 相当于js的filter,对每个对象进行遍历
```
GET /posts?title=json-server&author=Lorry
GET /posts?id=1&id=2
GET /comments?author.name=Lorry
```

- Paginate 分页功能 使用 `_page` 和可选的 `_limit`(默认10条)

```
GET /posts?_page=7
GET /posts?_page=7&_limit=20
```

response中会包含 `Link` 属性,表示第一个,下一个,和最后一个的链接,形如:
```
Link: <http://localhost:3000/todos?_page=1&_limit=1>; rel="first", <http://localhost:3000/todos?_page=2&_limit=1>; rel="next", <http://localhost:3000/todos?_page=5&_limit=1>; rel="last"
```

- Sort 使用 `_sort`(待排序类型) 和 `_order`(默认升序)

```
GET /posts?_sort=views&_order=asc
GET /posts/1/comments?_sort=votes&_order=asc
```
多个属性排序,使用 `,` 隔开
```GET /posts?_sort=user,views&_order=desc,asc```

- Slice `_start, _end, _limit` 使用此语法会在response中包含 `X-Total-Count` 表示所有的数量.注意,不包含end, `_gte,_lte, _lt, _gt, _ne` 表示>= , <= , >, <, ≠

```
GET /posts?_start=20&_end=30
GET /posts/1/comments?_start=20&_end=30
GET /posts/1/comments?_start=20&_limit=10
GET /posts?views_gte=10&views_lte=20
```

- 搜索

    - _like正则取值`GET /posts?title_like=aa$`

    - 全局搜索`GET /posts?q=internet`

- 上下级关系 `_embed, _expand` 前者添加子资源, 后者添加父资源,两者是相对的, comments通过postId(注意不是postsId)将post设为自己的父级,所以comments可以`_expand` post(注意不是posts),posts可以`_embed` comments
```
GET /posts?_embed=comments
GET /comments?_expand=post
```

- Database 获取数据库 `GET /db`

- Homepage 默认 index 文件 或者是 `./public` 目录也可以是自定义目录`json-server db.json --static ./some-other-dir`

## 自定义路由

- 创建routes.json文件,路径必须以`/`开头

```json
{
  "/api/*": "/$1",
  "/:resource/:id/show": "/:resource/:id",
  "/posts/:category": "/posts?category=:category",
  "/articles\\?id=:id": "/posts/:id"
}
```

- 启动时加入router选项 `json-server db.json --routes routes.json` 即可访问自定义地址

```
/api/posts              # → /posts 
/api/posts/1            # → /posts/1 
/posts/1/show           # → /posts/1 
/posts/javascript       # → /posts?category=javascript 
/articles?id=1          # → /posts/1
```

## Module 模块 (基于Express)

``` js
// server.js
const jsonServer = require('json-server')
const server = jsonServer.create()
const router = jsonServer.router('db.json') // 如果想用于内存储存的数据库,可直接使用jsonServer.router()
const middlewares = jsonServer.defaults()
 
server.use(middlewares)
server.use(router)
server.listen(3000, () => {
  console.log('JSON Server is running')
})
```

使用模块进行自定义路由

```js
const jsonServer = require('json-server')
const server = jsonServer.create()
const router = jsonServer.router('db.json')
const middlewares = jsonServer.defaults()
 
server.use(middlewares)
 
// 在use(router)前添加自定义路由
server.get('/echo', (req, res) => {
  res.jsonp(req.query)
  console.log('res.jsonp(req.query): ', res.jsonp(req.query));
})
// 添加重载规则
server.use(jsonServer.rewriter({
  '/api/*': '/$1',
  '/blog/:resource/:id/show': '/:resource/:id'
}))

// 添加bodyparser解决post , patch, put等请求方式
server.use(jsonServer.bodyParser)
server.use((req, res, next) => {
  if (req.method === 'POST') {
    req.body.createdAt = Date.now() // 获取创建时间
  }
  if(isAutorized(req)) {
      next() // 进行权限控制
  } else {
      // 自定义输出
    router.render = (req, res) => {
    res.status(500).jsonp({
    error: "error message here"
  })
}
  }
})
 
// 使用默认的路由
server.use(router)
// 挂载路由在/api上
server.use('/api', router)
server.listen(3000, () => {
  console.log('JSON Server is running')
})
```

- jsonServer.default([options])
    - static      静态文件路径
    - logger      true(default 下同)
    - bodyPaser   true
    - noCors      false
    - readOnly    只能使用GET请求 false


以上就是关于json-server的全部内容,需要一点node的知识,但还算好理解,在以后需要接口的时候便可以自力更生, 丰衣足食.







