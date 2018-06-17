---
layout:       post
title:        "React工具箱"
subtitle:     "工欲善其事,必先利其器"
date:         2018-06-11 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img: 'http://p799phkik.bkt.clouddn.com/paper.jpeg'
catalog:      true
multilingual: false
tags:
    - React
---
有一段时间没有更新博客了,之前的项目赶时间上线,所以加班很忙,在项目中使用了一段时间的react之后对其有了更深入的认识,现在就写一下写react都会用到哪些东西.有的是必备,有的是可以节省工作效率的东西.

## 1 node

node不用说,我们需要使用很多npm的模块,可以少造很多的轮子,也适合项目依赖的管理,结合ES6的语法,可以极大的免去我们管理依赖的烦恼.

### 最重要的就是package.json这个文件,包含三部分内容
1 包的基本属性
- names*
- version*
- description
- keyword
- homepage
- bugs
- license

2 用户相关信息
- author
- constributors

3 包中文件及模块描述
- main* 模块入口文件相对路径(模块根目录)
- files
- scripts
- config
- dependencies
- 还有很多描述属性,但最常用的为以上四种

"*"号表示必填项,可以使用npm init命令交互式创建package.json

### npm安装包的方式有三种

1 全局模式  --> -g 全局模块无法直接import或require,需要使用 ``` npm link <module name>``` 命令

2 本地模式  --> -S -D -O

- -S(--save) 安装dependencies的依赖 等同于-P(--save-prod)
- -D(--save-dev) 安装devDependencies的依赖
- -O(--save-optional)安装可选的依赖(optionalDependencies)

### npm的默认地址在海外,影响访问速度,可修改为国内淘宝镜像.修改方式为:

1 ``` npm install -g cnpm --registry = https://registry.npm.taobao.org ```

2 新建.npmrc,添加行``` registry = https://registry.npm.taobao.org ``` **(推荐这种方式)**

3 还有一种添加别名的方式,比较麻烦和繁琐,以上两条可满足需求,如有兴趣可参考[淘宝镜像官网](https://npm.taobao.org/)

如需卸载,使用``` npm config delete registry ```命令

### 自定义脚本 在``` scripts ```中添加对应脚本名和脚本

- npm为每个脚本默认携带了两个钩子,一个`pre-`,一个`post-`
- npm 默认的简写方式只有两个,一个是`npm start` 另一个是 `npm test` 其余都必须写上`run`

### 其他命令:

- `npm outdated` 检查包是否以过期
- `npm update module_name` 更新包
- `npm update -g npm` 更新npm本身
- `npm uninstall module_name` 卸载node模块
- `npm rebuild module_name` 用于更改包后重建
- `npm search module_name` 查找包是否存在(安装或发布时)
- `npm publish` 将模块发布到www.npmjs.org上,发布之前须进行注册并且使用`npm adduser`添加账号

一个值得另外一提的命令:

- `export DEBUG=module_name` 用于看到该module中的所有调试信息,在程序调试中相当有用,因为这样就比黑盒稍微白了那么一丢丢

## 2 ESLint

代码规范,必不可少,特别是多人协同,使代码更加可读且少bug

- 语法错误校验
- 多余或丢失的标点
- 不能到达的标点,比如return之后的语句
- 未被使用的参数
- 漏掉的结束符,如}
- 样式的统一 saas或less
- 检查变量命名

### 安装及初始化:

- `npm install -g eslint` 建议全局安装,因为几乎所有的前端环境都会用到
- `npm --init` 在工程根目录输入,按照要求进行填写即可,建议保存形式为.json,采用规范为airbnb

### 使用

- `eslint example.js` 或指定配置文件 `eslint -c config.json example.js`
- `"quotes": [2, "double"]` 配置规则
    - 第一部分是规则名,此处表示引号的规则
    - 第二部分表示级别,0 - 不验证 1 - 警告 2 - 错误
- 配置方法:
    - .eslint
    - package.json中添加eslintConfig模块
    - 代码文件中定义
- 关闭eslint的检查,开始部分添加注释`eslint-disble`,结束部分添加注释`eslint-enable`, 也可制定详细规则`eslint-disable no-console no-alert` ,`eslint-enable no-alert`, `eslint-enable no-console`分步定义规则
- react的检查,通过`eslint-plugin-react`实现,在`eslint --init`时回答需要使用react会自动进行配置和安装

## Babel 工具

让代码随处可读,具有最大的兼容性.

### 配置`.babelrc` 放在项目根目录下
- `"preset": []`转译规则,包含
    - `babel-preset-es2015` es6
    - `babel-preset-react` react
    - `babel-preset-stage-0` es7阶段0
    - `babel-preset-stage-1` es7阶段1
    - `babel-preset-stage-2` es7阶段2
    - `babel-preset-stage-3` es7阶段3
    - 不同阶段表示不同支持程度,一般采用stage2
    - 每种规则添加后都要进行安装 `npm install babel-preset-es2015 -D`安装到开发依赖中
- `"plugin": []` 配置各种插件

### 其他相关模块
- 命令行转译工具 babel-cli,用于通过命令行完成转码
    - `npm install -g babel-cli`
    - `babel [options] <files ...>`
        - optiond对应有 
            - -f --filename 读取输入时使用的文件名,通常被用于source map, errors
            - -o --out-file `babel -o result.js example.js` 将 `example.js` 转译为 `result.js`
            - -d --out-dir `babel src -d lib`对`src`目录下所有文件进行转译到输出目录`lib`
            - -s 生成source map文件 `babel src -d lib -s`(source map可将压缩后的代码有源可寻)
- 实时转译模块  `babel-register`
    - `npm install babel-register -D`
    - 先加载该模块,再引入其他模块即可实现实时
        - `require('babel-register')`
        - `require('example.js')`
- 浏览器实时转译模块 `browser.js`
    - 将其单独嵌入到html中
    - `<script src="browser.js"></script>` `<script type="text/babel">编写es6代码</script>`
    - 注意第二个script标签类型为text/babel
- 扩展转译模块 `babel-polifill`
    - babel只对es6的语法进行转译,不会对新增的api,如Maps, Iterator, Proxy, Generator,Symbol,Promise等进行转译,需要该模块
- babel-eslint 代码风格检查时也有可能使用到es6的语法.
    - 在.eslint添加`"parser": "babel-eslint","rules: {...}"`
- babel-core/register Mocha测试时对es6进行转译
    - `mocha --compiler js:babel-core/register`规定后缀名为js的文件都需要使用该模块编译


## Webpack 

一个供浏览器环境使用的打包工具,对静态资源进行统一管理.

- 安装 `npm install -g webpack`建议全局安装
- `webpack --config <custom config>`
- 配置文件: webpack.config.js
    ``` javascript
    module.exports = {
        entry: ['./app/main.js'],           // 入口文件
        output: {                           // 输出位置
            path: __dirname + '/build/',    // 文件存放绝对路径
            publicPath: '/build/',          // 项目运行时访问路径
            filename: 'bundle.js'           // 打包后文件名
        }
    }
    ```
- 如果文件包含es6和jsx语法,需要进行转译,用到之前提到的babel,webpack有专用的babel-loader和Bable预处理器
    ```
    npm install babel-loader -D
    npm install babel-preset-react -D
    npm install babel-preset-es2015 -D
    ```
    在webpack.config.js中添加
    ``` javascript
    module: {
        loaders: [{
            loader: "babel-loader",                  // 加载器
            text: "/\.jsx?$/",                       // 对应什么具体格式的文件        
            query: {presets: ['react', 'es2015']     // *.loader的参数
        },{
            loader: "style-loader!css-loader",       // 感叹号表示级联
            test: "/\.(css)$/"
        },{
            loader: "url-loader?limit=8129",         // 对小于9129byte的图片进行打包,一定程度上可以替代雪碧图
            test: "/\.(png|jpg)$/"
        },{
            loader: "style-loader!css-loader!less-loader",
            test: "\/.(less)$\"
        }
        ]
    }
    ```

- dev-server
    - `npm install -g webpack webpack-dev-server` 建议全局安装
    - 使用`webpack-dev-server` 便可直接启动开发服务器
    - 通常结合`react-hot-loader`实现开发时的热加载

- 热加载器(Hot Module Replacement HMR)
    - react每次更新的全局刷新虚拟DOM机制.
    - `npm install react-hot-loader -D`
    - 具体示例:
    server.js
    ``` js
    var webpack = require('webpack');
    var WebpackDevServer = require('webpack-dev-server');
    var config = require('../webpack.config');
    new WebpackDevServer (webpack(config), {
        publicPath: config.output.publicPath,
        hot: true,
        noInfo: false,
        historyApiFallback: true
    }).listen(3000,'127.0.0.1'), (err,res) => {
        err && console.log(err);
        console.log('Listening at localhost:3000');
    })
    ```
    webpack.config.js
    ``` js
    entry: [
        'webpack-dev-server/client?http://localhost:3000', // 需要与server中的监听地址一致
        'webpack/hot/only-dev-server',
        './script/entry'
    ]
    ```
    在webpack.config.json中添加loader
    ``` js
    loaders: [{
        test: "/.(js)$/",
        loader: 'react-hot!babel-loader?presets[]=react',
        exclude: /node-modules/,
    }]
    ```
    最后在package.json中加入脚本
    ``` json
    "script": {
        "start": "node ./js/server.js"
    }
    ```

- 打包成多个资源文件,如果把所有文件都打包放在一起进行加载会导致加载过程阻塞,影响用户体验,而且影响浏览器缓存,将不会经常改变的文件和会经常改变的文件分别打包,会优化浏览器速度
    ``` js
    {
        entry: {
            a: "./a",
            b: "./b"
        },
        output: {
            filename: "[name].js",  // 使用方括号name获取文件名
        },
        plugin: {
            new webpack.CommonChunkPlugin("common.js"); // 指定公共(public)文件夹下的打包为common.js
        }
    }
    // 总共会打包成为三个文件: a.js b.js common.js
    ```
- webpack的按需加载
    ``` js
    require.ensure(['module-a','module-b'], function(require) {
        let a = require('module-a');
    })
    // 执行到这段代码时可以按需加载
    
    ```

## 一个比较完整的项目目录结构
```
.
├── app
│   ├── components --> 组件
│   ├── imgs --> 图片资源
│   ├── index.html --> 入口
│   ├── main.css --> style
│   ├── main.jsx --> 主js
│   └── stores --> redux所需的store
├── node_modules
├── package.json --> 项目配置文件
├── test --> 测试用例
├── webpack.config.js --> 开发环境webpack
└── webpack.production.config.js --> 生产环境webpack
```



