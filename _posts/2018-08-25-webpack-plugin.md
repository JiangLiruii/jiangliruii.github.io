---
layout:       post
title:        "webpack plugin"
subtitle:     "webpack的毛细血管"
date:         2018-08-25 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/back/webpack-plugin.png'
catalog:      true
multilingual: false
tags:
    - web
---
# 概述
## webpack之所以强大是因为有许多的插件供其使用, 能够极大的丰富其功能.下面就重点介绍一些我工作中所用到的插件们, 基于版本 webpack 4.0

## SplitChunksPlugin

他的前身是`CommonsChunkPlugin`.该插件的默认配置即可满足大多数用户需求

### 默认配置

只会影响按需加载, 因为改变最初的chunks将影响HTML文件包含的script标签.
比如`import xxx from xxx`可以使用, 而`<script src="xxx">`不会生效

#### webpack将自动基于以下条件对chunks进行分割
- 新的chunk可以被共享, 或者来自node_modules下的modules, 即库代码与业务代码的分离
- 新的chunk比30kb大
- 当按需加载chunks的最大请求数量小于等于5
- 当初始page加载的最大请求数量小于等于3
当最后两个条件满足时, 尽可能设置大的chunks

### 自定义配置
```js
module.exports = {
    optimization: {
        //满足下列条件的都需要切分
        splitChunks: {
        // 对所有模块进行优化, 其余可选值有: async(异步加载块), initail(初始值, 经过测试跟'all'没什么区别)
        chunks: 'all',
        // 最小chunk化size, 为了产生chunk
        minSize: 20000,
        // 最大chunk化size, 如果大于这个值会进一步split(此处设置是1m), 不过如果原本一个模块超过了1m, 那么是不会对单个模块再进行切分的, 应该在HTTP/2 或者long_term cache中使用, 他会为了更好的换成(颗粒度变小)增加请求次数, 可以为了快速重构降低文件大小
        maxSize: 1000000,
        // 在切分前, 可以共享一个模块的最小chunks数量
        minChunks: 1,
        // 最大异步请求
        maxAsyncRequests: 5,
        // 入口的最大请求数
        maxInitialRequests: 3,
        // 自定义分隔符将原有名 + chunk名(比如: vendor~main.js)
        automaticNameDelimiter: '~',
        // true根据模块和缓存组关键字自动生成, 也可以是name(module){return}, 命名需唯一.
        name: true,
        // cacheGroup优先级最高, 可覆盖之前的所有属性, 关闭cache groups 可设置{cacheGroups: {default: false}}
        cacheGroups: {
            venders: {
                // 控制被选择的模块, 如果选择所有模块就忽略掉它, 它能够匹配绝对模块原路径或者chunk名, 当chunk名被匹配,在这个chunk里的所有模块都会被选中.
                test: /[\\/]node_modules[\\/]/,
                priority: -10
            },
            react: {
                rest: /react/,
                priority: -9
            }
            default: {
                minChunks: 2,
                // 一个模块可以属于多个缓存组, 优化器会倾向于更高 priority属性的缓存组, 默认组通常priority为负, 因为自定义组的默认值为0
                priority: -20,
                // 如果当前chunk包含的module已经被main bundle切分, 那么会重复使用而不是新生产一个chunk, 会影响依赖chunk名产生的文件名
                reuseExistingChunk: true,
            }
        }
    }, 
}
```
属性的优先级排序:
`maxInitialRequest/maxAsyncRequests < maxSize < minSize`