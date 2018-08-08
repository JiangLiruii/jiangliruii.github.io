---
layout:       post
title:        "webpack中的css-loader和style-loader"
subtitle:     "网页三剑客中的css是的打包对网页的呈现以及编码十分重要"
date:         2018-08-07 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/compare.jpg'
catalog:      true
multilingual: false
tags:
    - Webpack
---

# CSS Loader
css loader 会解析 类似于`import/require` 的`@import` 和 `url()`
 
## install 
`yarn add css-loader -D`
## Usage
一个好的用于require assets的loader是 `file-loader` 和 `url-loader`

```js
// file.js
import css from 'file.css'
// webpack.config.js
module.exports = {
    module: {
        rules: [
            test: /\.css$/,
            use: ['style-loader', 'css-loader']
        ]
    }
}
```
### options

Name|Type|Default|Description
---|---|---|---
url|{Boolean}|true|Enable/Disable url() handling
import|{Boolean}|true|Enable/Disable @import handling
modules|{Boolean}|false|Enable/Disable CSS Modules
localIdentName|{String}|[hash:base64]|Configure the generated ident
sourceMap|{Boolean}|false|Enable/Disable Sourcemaps
camelCase|{Boolean|String}|false|Export Classnames in CamelCase
importLoaders|{Number}|0|Number of loaders applied before CSS loader

下面重点讲一下option中的modules
#### modules: enable **CSS Modules** spec

该参数开启了local scope CSS, 可以分别在selector和rules中使用`:glocal(...)` 或者`:global`.

- Scope
默认CSS向全局selector scope导出所有的classnames, 全局注定就造成了污染,主要影响到每个classname要唯一, 并且尽可能在小范围selector(避免使用标签选择器), 减小在全局中的影响.

Styles可以本地化来避免这个问题.

`:local(.className)`可以让此className在local scope, `local` 也会被导出

`:local`没有括号表示启用了本地模式

同理于 `:glocal(.className)` 和`:global`

#### Composing 组合

通过不同的classnames进行组合, 实现多classname的导出, 例如
```css
:local(.className) {
  background: red;
  color: yellow;
}

:local(.subClass) {
  composes: className;
  background: blue;
}
```
导出为:
```js
exports.locals = {
  className: '_23_aKvs-b8bW2Vg3fwHozO',
  subClass: '_13LGdX8RMStbBE9w-t0gZ1 _23_aKvs-b8bW2Vg3fwHozO'
}
```
css为:
```css
._23_aKvs-b8bW2Vg3fwHozO {
  background: red;
  color: yellow;
}

._13LGdX8RMStbBE9w-t0gZ1 {
  background: blue;
}
```

#### Importing 
从其他module中导入本地classname
```css
:local(.continueButton) {
    composes:button from 'library/button.css';
    background: red;
}

:local(.nameEdit) {
    composes: edit hightlight from './edit.css';
    background: red;
}
```
还可以导入多条rules
```css
:local(.className) {
    composes: edit hightlight from './edit.css';
    composes: button from 'module/button.css';
    composes: classFromThisModule;
    background: red;
}
```

#### localIdentName
可以通过`localIdentName`配置生成的ident(标识)
```js
// webpack.config.js
{
    test: /\.css$/,
    use: [
        {
            loader: 'css-loader',
            options: {
                module: true,
                localIdentName: '[path][name]__[local]--[hash:base64:5]'
            }
        }
    ]
}
```
#### sourceMap
默认的sourceMap功能是没有开启的,因为会增大编译时间和编译后的bundle.js的体积, 另外相对路径会导致bug, 需要使用包含`server URL` 的绝对public路径, 
```js
// webpack.config.js
{
    loader: 'css-loader',
    options: {
        sourceMap:true
    }
}
```
### camelCase
默认情况导出的JSON keys与class names互为镜像, 如果希望使用驼峰命名, 就需要配置此参数
Name|Type|Description
---|---|---
true|{Boolean}|所有classname 被驼峰化
'dashes'|{String}|只有classname中包含'-'被驼峰化
'only'|{String}|classname被驼峰化,原classname从locals中丢弃
'dashesOnly'|{String}|只有classname中包含'-'被驼峰化,原classname从locals中丢弃

举个例子:
```js
// file.css
.class-name {}
// file.js
import {className} from 'file.css'

// webpack.config.js
{
    loader: 'css-loader',
    options: {
        camelCase:true
    }
}
```

#### importLoaders

参数 `importLoaders`允许配置在css-loader前通过@import应用多少个loaders.
```js
// webpack.config.js
{
    test: /\.css$/,
    use: [
        'style-loader',
        {
        loader: 'css-loader',
        options: {
            importLoaders: 2 //默认值0 表示没有loader, 1 表示一个, 2 表示两个, 以此类推
        }
    },
    'postcss-loader',
    'sass-loader']
}