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

```js
// basic function
const slice = Array.prototype.slice
```
```js
// transform generator to promise
function co(gen) {
    let ctx = this;
    let args = slice.call(arguments, 1);
    // wrap everything in promise
    return new Promise((resolve, reject) => {
        if (typeof gen === 'function') gen = gen.apply(ctx, args);
        if(!gen || typeof gen.next !== 'function') resolve(gen);
        onFulfilled();
    })
    function onFulfilled(res) {
        let ret;
        try {
            ret = gen.next(res);
        } catch(e) {
            return reject(e);
        }
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
    function next(ret) {
        if(ret.done) return resolve(ret.value);
        let value = toPromise.call(ctx, ret.value);
        if(value && isPromise(value)) return value.then(onFulfilled, onRejected);
        return onRejected(new TypeError(`You may only yield a function, promise, generator, array,or object, but the following object was passed ${String(ret.value)}`))
    }
}

// Convert a 'yield'ed value into a promise
function toPromise(obj) {
    if (!obj) return obj;
    if (isPromise(obj)) return obj;
    if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);
    if (typeof obj === 'function') return thunkToPromise.call(this, obj);
    if (Array.isArray(obj)) return arrayToPromise.call(this, obj);
    if (isObject(obj)) return objectToPromise.call(this, obj);
    return obj;
}

// Convert a thunk to promise

function thunkToPromise(fn) {
    let ctx = this;
    return new Promise((resolve, reject) => {
        fn.call(ctx, (err, res) => {
            if(err) return reject(err);
            if(arguments.length > 2) res = slice.call(arguments, 1);
            resolve(res)
        })
    })
}

// Convert a array to promise

function arrayToPromise(arr) {
    return Promise.all(arr.map(toPromise, this))
}

// Convert object to promise

function objectToArray(obj) {
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