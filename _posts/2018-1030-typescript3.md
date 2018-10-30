---
layout:       post
title:        "typescript 3.1"
subtitle:     "typescript 3.1 简介"
date:         2018-10-30 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/performanceEvent.jpg'
catalog:      true
multilingual: false
tags:
    - web
---
# 在工程应用中, typescript确实要比js好用很多, 虽然要多写类型判断的代码, 但是这都是前人栽树后人乘凉的好事. 现在ts 3.1发布, 是应该好好学学.

## ts3.0介绍了一种新的项目引用的概念, 项目引用可以让ts项目依赖于另一个ts项目, 特别是允许 `tsconfig.json`文件依赖于另一个`tsconfig.json`. 因为它给了ts(以及与ts相关的工具)一种去理解构建顺序和输出结构的方式, 细化这些依赖可以更轻松的将代码分离为一个个小项目.

## ts3.0 为tsc介绍了一种新技术: `--build` 标志位, 它将帮助实现更快的ts构建中的项目引用.

### Example Project

#### 假设有如下结构的工程, 有两个模块, 一个convert, 一个unit, 文件分别为:
```text
/src/converter.ts
/src/units.ts
/test/converter-tests.ts
/test/units-tests.ts
/tsconfig.json
```

#### 这些test文件导入了implementation文件并且做了一些测试,
```ts
// converter-test.js
import * as converter from "../convert";

assert.areEqual(converter.celsiusFahrenheit(0), 32);
```
#### 从前因为只有一个tsconfig文件, 这些结构会相当的笨拙
- implementation文件有可能import test文件
- 有可能同时构建 `test` 和 `src` 但是在输出的文件夹名字涨没有你想要的`src`.
- 仅仅改变内部的implementation文件需要再一次 类型检测 ts文件, 尽管它不会造成新的错误
- 仅仅改变tests需要进行类型再次的检测, 尽管没有任何接口被改变

#### 使用多个tsconfig可以解决一些上述的问题, 但是会有额外的问题在等着你
- 没有内建的更新检查, 所以最终需要run两次 `tsc`
- 触发`tsc`两次产生更多的初始化时间
- `tsc -w`不能在多个config文件中同时运行

### Project Reference 项目引用可以解决以上所有问题

### 什么是Project Reference?

#### `tsconfig.json`文件有一个新的最高级属性,, `reference`, 它是一个对象数组, 描述项目引用.
```json
{
    "compilerOptions": {
        // the usual
    },
    "reference": [{
        "path": "../src"
    }]
}
```
#### 每一个引用的`path`属性都可以只想一个具体包含`tsconfig.json`的路径,或者指向具名的配置文件本身

#### 当你引用了一个项目时, 新的事情发生了
- 从引用中引入模块将加载它的输出声明文件(`.d.ts`)
- 如果引用项目生成了`outFile`, 这个`.d.ts`文件的声明将对当前项目可视
- 构建模式会自动在需要时构建引用项目(而不是每次改变时)

通过分离为多个项目, 我们可以极大的提高类型检查或编译的速度, 减少使用编辑器时的内存占用, 提高在程序中逻辑归类的有效性.
### composite