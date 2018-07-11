---
layout:       post
title:        "Event Loop"
subtitle:     "我不管你应不应,我会永远的询问你,只为万一某个瞬间的你需要"
date:         2018-07-04 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/non-blocking-1.png'
catalog:      true
multilingual: false
tags:
    - JavaScript
---

事件轮询是js中最重要的部分之一,只有理解了这个东西,才能理解为什么有callback,为什么用单线程.

# [HTML标准](https://html.spec.whatwg.org/#event-loops)里的定义

为了定位事件,用户交互,脚本,渲染, 网络等等,user agent使用事件轮训来描述这些部分. 有两种轮询:

- for browsing context
- for workers

在每个user agent中至少有一个browsing context 类事件轮询,并且在每个[similar-origin browsing  context](https://html.spec.whatwg.org/#origin)中最多只有一个

每个worker有一个event loop, 并且这个worker处理模型管理事件轮询的生命周期

一个事件轮询有一个或多个task queues, task queues是一个tasks的有序列表(宏任务), 为下列类型的算法:

- Event
- Parsing: HTML parser
- Callbacks
- Using a resource
- Reacting to DOM manipulation

每个事件轮询都有一个微任务(microtask)

- 独立的回调(solitary callback microtasks)
  - 当user agent到达微任务检查点(microtask checkpoint)时, 如果标志位为false,则会往下执行
  - 设置标志位为true
  - while(microtask not empty)
    - 将最老的微任务(oldestMicrotask)成为最老的微任务
    - 设置当前任务为该最老的任务
    - 运行最老的任务
    - 设置当前运行任务为空
    - 从microtask中移除最老的任务
  - 为每个响应事件循环为本次循环的环境设置对象(environment settings object)并通知拒绝的promises
  - 清除索引的数据库交换(cleanup indexed database transactions)
  - 设置微任务检查点的标志为false
- 复合微任务(compound microtasks)
  - 让parent成为事件轮询中当前事件(当前运行compound microtask)
  - 让subtask成为新的由正在运行的系列任务组成的事件
  - 设置事件轮询的当前任务为substack

宏任务和微任务的具体实现:
- macrotasks: setTimeout setInterval setImmediate I/O UI渲染
- microtasks: Promise process.nextTick Object.observe MutationObserver
## 千字不如一图
### 下面是event-loop的运行流程

![event-loop](http://p799phkik.bkt.clouddn.com/event-loop-new.png)

### 下面是一个例子,展示macrotask和microtask的运行原理.
![execute-process](http://p799phkik.bkt.clouddn.com/execute-process.png)