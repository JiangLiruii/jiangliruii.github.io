---
layout:       post
title:        "CO Library"
subtitle:     "about Generator"
date:         2018-07-12 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/promise.jpg'
catalog:      true
multilingual: false
tags:
    - Interesting
---

## CO库是用于将generator转换成promise, 实现跨浏览器兼容的极其简洁和清爽的利器. 之前深入探讨过promise, 所以想继续揭开generator这个在js中另一个重要的异步语法.
原github地址: https://github.com/tj/co

直接上代码,一行一行分析
```js
// 切分参数
const slice = Array.prototype.slice
// 将generator转化成promise
function co(gen) {
    // 保存上下文
    let ctx = this;
    // 获取剩余参数, ES6 可支持...args
    let args = slice.call(arguments, 1);
    // 返回新的Promise
    return new Promise((resolve, reject) => {
        // 如果gen为函数,则进行调用
        if (typeof gen === 'function') gen = gen.apply(ctx, args);
        // 如果没有gen或者gen没有next方法, 则将gen resolve掉, 可在co(gen).then(res)中获取
        if(!gen || typeof gen.next !== 'function') resolve(gen);
        // 都不是的话则显式调用成功方法
        onFulfilled();
    })
    // 定义成功方法
    function onFulfilled(res) {
        // 外部'返回'变量
        let ret;
        // 使用try-catch捕捉next时发生的错误
        try {
            // 将res传给next方法(gen中的a = yield 变量中)
            ret = gen.next(res);
        } catch(e) {
            return reject(e);
        }
        // 调用next方法
        next(ret);
        return null;
    }

    function onRejected(err) {
        let ret;
        try {
            ret = gen.throw(err)
        } catch(e) {
            return reject(e)
        }
        next(ret);
    }
    // 检验gen.next返回值
    function next(ret) {
        // 如果gen已经完成遍历
        if(ret.done) return resolve(ret.value);
        // 否则将value转化成Promise
        let value = toPromise.call(ctx, ret.value);
        // 如果有value, 且value的为promise, 会自动调用value的resolve, 然后会在微任务中调用onFufilled, 和onRejected
        if(value && isPromise(value)) return value.then(onFulfilled, onRejected);
        // 否则拒绝
        return onRejected(new TypeError(`You may only yield a function, promise, generator, array,or object, but the following object was passed ${String(ret.value)}`))
    }
}

// Convert a 'yield'ed value into a promise
function toPromise(obj) {
    // 如果obj为空,则返回
    if (!obj) return obj;
    // 如果obj本身就是promise,直接返回
    if (isPromise(obj)) return obj;
    // 如果obj是generator, 则再调一次co函数
    if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);
    // 如果obj时一个不为generator的函数, 将函数转化为promise
    if (typeof obj === 'function') return thunkToPromise.call(this, obj);
    // 如果obj是列表,将列表转为promise
    if (Array.isArray(obj)) return arrayToPromise.call(this, obj);
    // 如果obj是对象, 将对象转为promise
    if (isObject(obj)) return objectToPromise.call(this, obj);
    return obj;
}

// 将函数转化为promise

function thunkToPromise(fn) {
    let ctx = this;
    return new Promise((resolve, reject) => {
        // 调用fn,
        fn.call(ctx, (err, res) => {
            if(err) return reject(err);
            if(arguments.length > 2) res = slice.call(arguments, 1);
            resolve(res)
        })
    })
}

// 将array转化为Promise

function arrayToPromise(arr) {
    return Promise.all(arr.map(toPromise, this))
}

// 将object转为Promise

function objectToPromise(obj) {
    // make one pure obj which will convert later
    let results = new obj.constructor();
    let keys = Object.keys(obj);
    let promises = [];
    for (let key of keys) {
        let promise = toPromise.call(this, obj[key]);
        if (promise && isPromise(promise)) defer(promise, key);
        else results[key] = obj[key];
    }
    return Promise.all(promises).then(()=>results);
    function defer(promise, key) {
        // pre undefined the key in result
        results[key] = undefined;
        promises.push(promise.then(res => results[key] = res))
    }
}

function isPromise(obj) {
    return typeof obj.then === 'function';
}

function isGenerator(obj) {
    return typeof obj.next === 'function' && typeof obj.throw === 'function';
}

function isGeneratorFunction(obj) {
    let constructor = obj.constructor;
    if(!constructor) return false;
    return isGenerator(constructor.prototype)
}

function isObject(val) {
    return Object === val.constructor;
}
```