---
layout:       post
title:        "webpack中的css-loader和style-loader"
subtitle:     "网页三剑客中的css是的打包对网页的呈现以及编码十分重要"
date:         2018-08-07 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/webpack_css.jpg'
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
```
# postcss-loader,这是一个可以有丰富自定义功能的css打包工具

## install `yarn postcss-loader -D`

## configuration 每个css目录下都可以新建一个`postcss.config.js`
```js
module.exports = {
    parser: 'sugarss',
    plugins: {
        'postcss-import':{},
        'postcss-preset-env': {},
        'cssnano': {}
    }
}
```
需用在css-loader之后, sass/less等之前.
```js
// webpack.config.js
module.exports = {
    modules: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    {
                        loader: 'css-loader',
                        options: {
                            importLoader: 1
                        }
                    },
                    'postcss-loader',
                ]
            }
        ]
    }
}
```
## Options
Name|Type|Default|Description
---|---|---|---
exec|{Boolean}|undefined|Enable PostCSS Parser support in CSS-in-JS
parser|{String|Object}|undefined|Set PostCSS Parser
syntax|{String|Object}|undefined|Set PostCSS Syntax
stringifier|{String|Object}|undefined|Set PostCSS Stringifier
config|{Object}|undefined|Set postcss.config.js config path && ctx
plugins|{Array\|Function}|[]|Set PostCSS Plugins
sourceMap|{String|Boolean}|false|Enable Source Maps

### exec 如果没有添加`postcss-js `parser的话, 需要添加 `exec`为true
```js
// postcss 源码
    if (options.parser === 'postcss-js') {
      css = this.exec(css, this.resource)
    }

    if (config.exec) {
      css = this.exec(css, this.resource)
    }
```
如果parser中没有postcss-js, 那么可以设置exec为true, 效果一样.
### Config
Name|Type|Default|Description
--|--|--|--
path|{String}|undefined|PostCSS Config Directory
context|{Object}|undefined|PostCSS Config Context
### path 
可手动设置config的搜索路径, 当将config文件分别储存的时候需要.但是需要注意的是必须将配置文件的名字设置为 `.postcss.js 或 postcss.config.js`, 此处设置的路径只能到文件夹
```js
{
  loader: 'postcss-loader',
  options: {
    config: {
      path: 'path/to/.config/' ✅
      path: 'path/to/.config/css.config.js' ❌
    }
  }
}
```
### Context(ctx)
Name|Type|Default|Description
--|--|--|--
env|{string}|'development'|process.enc.NODE_ENV
file|{Object}|loader.resourcePath|extname,dirname, basename
options|{Object}|{}|Options

`postcss-loader` 暴露上下文文给config 文件, 可以使得配置文件动态配置
```js
// postcss.config.js
module.exports =({file, options, env}) => ({
    parser: file.extname === '.sss' ? 'sugarss': false,
    plugins: {
        'postcss-import': {root: file.dirname},
        'postcss-preset-env': options['postcss-preset-env'] ? options['postcss-preset-env'] : false,
        'cssnano': env === 'production' ? options.cssnano: false,
    }
})
// webpack.config.js
{
    loader: 'postcss-loader',
    options: {
        config: {
            ctx: {
                'postcss-preset-env': {...options},
                cssnano: {...options},
            }
        }
    }
}
```

## Plugins
```js
// webpack.config.js
{
    loader: 'postcss-loader',
    options: {
        idet:'postcss',
        plugins (loader) => [
            require('postcss-import')({root: loader.resourcePath}),
            require('postcss-preset-env')(),
            require('cssnano')()
        ]
    }
}
```
## Syntaxes
Name|Type|Default|Description
--|--|--|--
parser|{String|Function}|undefined|Custom PostCSS Parser
syntax|{String|Function}|undefined|Custom PostCSS Syntax
stringifier|{String|Function}|undefined|Custom PostCSS Stringifier

### parser 之前已经配过的`options:{parser: 'sugarss'}`
### syntax `options:{syntax: 'sugarss'}`
### stringifier `options:{stringifier: 'midas'}`

## 使用案例
### 1 如果想在js中写css
```js
// webpack.config.js
{
  test: /\.style.js$/,
  use: [
    'style-loader',
    { loader: 'css-loader', options: { importLoaders: 2 } },
    { loader: 'postcss-loader', options: { parser: 'postcss-js' } },
    'babel-loader'
  ]
}
// xx.style.js
import colors from './styles/colors'

export default {
    '.menu': {
        color: colors.main,
        height: 25,
        '&_link': {
            color:'white'
        }
    }
}
```
### 2 Extract css
```js
// webpack.config.js
const devMode = env.process.NODE_ENV !== 'production'
const MiniCSSExtractPlugin = require('mini-css-extract-plugin')
module.exports = {
    module: {
        rules: {
            test:/\.css$/,
            use: [
                devMode ? 'style-loader' : MiniCSSExtractPlugin.loader,
                'css-loader',
                'postcss-loader', 
            ]
        }
    },
    plugins: [
        new MiniCSSExtractPlugin({
            filename: devMode ? '[name].css' : '[name].[hash].css'
        })
    ]
}
```