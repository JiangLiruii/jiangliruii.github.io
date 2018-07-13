---
layout:       post
title:        "Function Program"
subtitle:     "Make Programe Functional"
date:         2018-07-10 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/promise.jpg'
catalog:      true
multilingual: false
tags:
    - JavaScript
---

## 一个小小的例子-- pipeline
```js
const pipe = functions => data => {
    return functions.reduce((value, fun) => fun(value))
}

const cart = [3.12, 45.15, 11.01]

const addTax = (total, taxRate) => total * (1 + total)

const totally = cart => pipe([
    x => x.reduce((t,v) => t+v),
    x => addTax(x, 0.09),
    x => `Order Total = ${x.toFixed(2)}`
])(cart);