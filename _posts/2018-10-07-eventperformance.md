---
layout:       post
title:        "优化事件处理函数"
subtitle:     "debounce throttle requestAnimationFrame"
date:         2018-10-07 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/performanceEvent.jpg'
catalog:      true
multilingual: false
tags:
    - web
---
# 概述
## 浏览器的性能是不同的, 如果事件触发时间较短, 频率较高, 必然会导致浏览器跟不上事件的频率, 导致掉帧,卡顿等问题, 用户体验不佳, 而且事件频繁触发会消耗更多的 cpu 资源以及电量, 对移动端来说是非常严苛的. 

### 所以就需要对事件的处理进行优化.主要有三个方法, 

- debounce, 函数防抖
- throttle, 函数节流
- requestAnimationFrame, 浏览器的新方法

## 先来看看最新且名字最长的** requestAnimationFrame **, 他是在 window 上的方法, 不用引入任何的第三方库, 且会根据重绘所需的时间来进行调用, 所以理论上来说不会造成任何的卡顿, 只是执行的次数会变少.

下面是一个例子, 模拟了一个进度条.

<p data-height="265" data-theme-id="0" data-slug-hash="dgONLL" data-default-tab="js,result" data-user="JiangLiruii" data-pen-title="dgONLL" class="codepen">See the Pen <a href="https://codepen.io/JiangLiruii/pen/dgONLL/">dgONLL</a> by Jiang Lirui (<a href="https://codepen.io/JiangLiruii">@JiangLiruii</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

js 代码为:
```js
var start = null;
var element = document.getElementById('SomeElementYouWantToAnimate');
element.style.position = 'absolute';

// 在 requestAnimationFrame 中接受一个时间戳的参数, 代表了执行函数时的当前时间
function step(timestamp) {
  if (!start) start = timestamp;
  var progress = timestamp - start;
  element.style['padding-left'] = Math.min(progress / 10, 500) + 'px';
  // 指定一个终点
  if (progress < 5000) {
    // 迭代调用该方法
    window.requestAnimationFrame(step);
  }
}

window.requestAnimationFrame(step);
```
上述方法是递归调用, 他的厉害之处还不在这里, 比如 scroll 事件的调用

<p data-height="265" data-theme-id="0" data-slug-hash="ReoVaZ" data-default-tab="js,result" data-user="JiangLiruii" data-pen-title="ReoVaZ" class="codepen">See the Pen <a href="https://codepen.io/JiangLiruii/pen/ReoVaZ/">ReoVaZ</a> by Jiang Lirui (<a href="https://codepen.io/JiangLiruii">@JiangLiruii</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

js 源码为:
```js
// 设置标志位
var ticking = false;
var outter = document.getElementById('outter')
outter.addEventListener('scroll', () => { 
    if(!ticking) {
        // 当重绘完成以后调用
        window.requestAnimationFrame(() =>{
        console.log('with', new Date());
        // 重置标志位
        ticking = false;
        })
    }
    // 将标志位设为 true, 阻止继续requestAnimationFrame的调用
    ticking = true;
})
// 参考不使用 requestAnimation 的情况
outter.addEventListener('scroll', () => {
  console.log('without', new Date());
})
```
### 使用 rAF 的好处是原生 API, 容易维护执行, 且精度较高, 基本上可以视为16ms 的 throttle, 当然缺点也比较明显, 只能在前端使用, 不支持后端, 太频繁的 rAF 调用仍需要通过资质 throttle 进行调节
## 再来一起看看函数节流和函数防抖, 节流就是从源头上就不让其发生, 防抖是让其发生但会忽略掉之前的一部分.

比如:
<p data-height="265" data-theme-id="0" data-slug-hash="WaojaB" data-default-tab="js,result" data-user="JiangLiruii" data-pen-title="WaojaB" class="codepen">See the Pen <a href="https://codepen.io/JiangLiruii/pen/WaojaB/">WaojaB</a> by Jiang Lirui (<a href="https://codepen.io/JiangLiruii">@JiangLiruii</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
js 源码为

```js
// 全局 timeout 变量
var t = null
var outter = document.getElementById('outter')
function debounce(func, timeout=1000) {
    if(!t) {
        // 设置异步执行
        t = setTimeout(() => {
            // 调用回调函数
            func()
            // 清除
            t = null
        }, timeout)
    } else {
        // 清除
        t = clearTimeout(t)
    }
}
outter.addEventListener('scroll', () => {
  debounce(() => {
      console.log('debounce')
  })
})
outter.addEventListener('scroll', () => {
  console.log('without debounce')
})
```

### 下面是 throttle, 其他代码都一样, 唯一不同的是针对已有定时器的处理方式, 节流就是不让后面的再发生了, 要让之前的定时器完成了之后才"开流"

```js
function throttle(func, timeout=1000) {
    if(!t) {
        t = setTimeout(() => {
            func();
            t = null;
        }, timeout)
    } else {
        // 这里是最主要的区别
        return
    }
}
```

### 以上就是最基础的版本了, 当然传入的函数还有其他的参数, 甚至 this 的绑定问题, 那么可以进行高阶函数的抽象

```js
function throttle(func, timeout=1000, ...args) {
    var t;
    // 返回一个函数
    return function() {
        if(!t) {
            t = setTimeout(() => {
                // 这里将 this 和参数一并传入
                func.apply(this,args);
                t = null;
            }, timeout)
        } else {
            return
        }
    }
}
outter.addEventListener('scroll', throttle(() => {
      console.log('throttle')
  }))
outter.addEventListener('scroll', () => {
  console.log('without throttle')
})
```

### 还有更进一步的需求, 比如 underscore 的 [throttle](https://underscorejs.org/#throttle),[debounce](https://underscorejs.org/#debounce)

- throttle  默认会在第一次时立即调用函数, 用 option 来设置 trail 和 leading.
- debounce有 immediate 参数来区分 trail 还是 leading 模式, 如果是 true 为 leading 模式, 即先触发函数, 之后同理, 等待 timeout, 在期间如果被打断, timeout 就会刷新, 重新计时.