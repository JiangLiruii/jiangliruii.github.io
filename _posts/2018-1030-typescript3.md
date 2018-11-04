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

#### 引用的项目必须有新的 `composite`设置并且为 enable 状态, 这个设置是用来保证 ts 能够快速决定从哪里去找到引用项目的输出, 使能 composite 标志位可以改变下面这些事情

- `rootDir`设置, 如果不是显式设置, 默认的会指向 `tsconfig`文件的目录
- 所有接口文件必须被`include`模式匹配, 或者列在`file`的数组中, 如果这些限制被违反, `tsc`会通知你哪一个文件没有具体化
- `declaration`声明必须被打开

### declarationMaps
####  我们也可以添加 `[declareation source maps](https://github.com/Microsoft/TypeScript/issues/14479)`. 如果你打开 `--declarationMap`, 你将会打开编辑器的类似于'Go to Definition'的特性, 并且可以在支持的 editor 中跨项目进行透明式导航以及变基代码

### prepend with outFile
#### 你也可以打开 在输出的依赖中prepend 属性
```
"reference": [{
    "path":"../utils",
    "prepend": true
}]
```
在项目中开头添加将包括在当前项目输出之上的项目输出, 这将包括`.js` 和 `.d.ts` 文件, 并且 sourcemap文件会被正确的忽略掉

`tsc`将紧紧对已存在硬盘中的文件进行这一过程, 所以可能创建一个项目之后, 却因为一些项目输出将被在结果文件中展示多次导致正确的输出文件无法生成, 类似于
```
    A
   ^ ^
  /   \
 B     C
  ^   ^
   \ /
    D
```
不在每个引用的头进行添加是非常重要的, 因为你讲最终得到两份副本在 D 的输出中, 这将导致不可预期的结果

## 项目引用的权衡
### 项目引用也有一些需要你去权衡的点
#### 因为独立的项目将会使用基于依赖的 `.d.ts` 文件, 你将不得不选择在克隆项目之后, 在可以使用编辑器去导航而不会有错误之前,检查某种构建好的输出或构建一个项目, 我们现在一直会工作在 .d.ts 产生的进程里,虽然他可以缓解这个问题, 但是还是建议开发者应该在克隆之后进行构建
#### 另外的还有为了与已有的构建工作流保持兼容性, `tsc`将自动构建依赖, 除非是通过 `--build` 触发, 下面就学学更多的关于 `--build` 的内容

### ts构建模式

#### 有一个期待很久的特性是为 ts 项目智能化递增构建, 在3.0中可以在` tsc` 中使用`--build` 标志位, 这对` tsc` 来说是一个有效的新入口, 可以做更多的事情比如把构建安排在一起而不仅仅是个编译器

#### 运行 `tsc --build(tsc --b)`会做以下的事情:
- 查找所有相关的项目
- 探测他们是否是最新的
- 按照正确的顺序构建过时的项目

#### 你可以为`tsc -b`提供多个配置文件地址, 比如`tsc -b src test`, 就像` tsc -p`一样, 如果配置文件本身就是 tsconfig.json, 那么可以不用具体化配置文件的名字.
#### `tsc -b`的命令行
```
 > tsc -b                                # Build the tsconfig.json in the current directory
 > tsc -b src                            # Build src/tsconfig.json
 > tsc -b foo/release.tsconfig.json bar   # Build foo/release.tsconfig.json and bar/tsconfig.json
 ```
#### 不用担心传过去的文件顺序, tsc 将会根据需要重新排序, 所以那些依赖总是最先被 build
#### 有这些 flag 可以给 `tsc -b`使用
- --verbose: 打印 verbose 日志来解释发生了什么
- --dry: 展示做了什么而不是实际构建的东西
- --clean: 删除特定项目的输出, 可以结合 --dry 使用
- --watch: 监听模式, 可以结合任意的除了--verbose 的 flag使用

### 权衡
#### 通常来说 ts会产生 .js 或.d.ts 的输出在当前的语法或类型错误里, 除非`noEmitOnError`是打开的.  这样做在一个增量构建系统里是非常恐怖的---如果你任意一个过时的依赖有新的错误, 你只能看见它一次因为构建子序列会跳过去构建最新项目, 因为这个原因当 `noEmitOnError`使能时, tsc -b 可以有效的在所有文件中执行

#### 如果你检查任意一个构建输出 .js, .d.ts, .d.ts.map 等等, 当某个资源控制操作依赖于资源控制工具是否保留时间戳在本地副本和远程副本之间, 你可能需要添加 `--force` 去构建.

### MSbuild
#### 如果你有一个 msbuild 项目, 你可以通过添加以下代码到 proj 文件中打开这个构建模式, 这将自动打开增量构建模式, 同时也很清真
```
    <TypeScriptBuildMode>true</TypeScriptBuildMode>
```
#### 请注意, 如果和 tsconfig.json / -p 一起的话, 已有的 ts 项目属性将不会被考虑, 所有的设置都将会使用 tsconfig 文件.

#### 一些团队有设置 msbuild-base 在tsconfig文件中 的工作流, 有一些同样隐式的图表顺序在匹配的项目管理中. 如果你的解决方案跟此类似, 你可以继续使用 msbuild 和 有项目引用的 tsc -p, 他们是彼此协作的

## 指引
### 大体框架

#### 通过和更多的 tsconfig.json 文件, 你可以使用遗传配置文件去集中化你的常用编译选项, 这将是你可以在一个文件中改变设置, 而不是去编辑多个文件

#### 另一个良好的实践是 tsconfig.json 文件仅仅有一个所有叶子节点项目的 reference , 这代表一个简单的入口, 比如, 在 ts 仓库里仅仅运行 tsc -b src 去构建所有的终点, 因为我们在 src/tsconfig.json 中列出了所有的子项目, 注意, 这是在3.0才开始的, 如果一个空的 files 数组不再会是错误, 如果你仅仅只有一个引用在 tsconfig.json 中

#### 你可以看到这些模式: src/tsconfig_base.json, src/tsconfig.json, and src/tsc/tsconfig.json

### 相关模块框架
#### 通常情况下, 一个仓库会包含的相关模块不会很多, 简单的将一个给出的父目录的每个子目录放进 `tsconfig`中, 并且给这些配置文件添加`reference`去匹配程序中对应的地方, 会需要设置`outDir` 到一个显式的输出文件夹下的子文件夹, 或者对所有共同根目录的所有项目文件夹设置 `rootDir`

### outFiles的结构
#### 编译 `outfile`的排版 是很灵活的, 因为相对路径并没有多大的影响, 需要记住的是不要在最后一个项目之前使用`prepend`, 这将会在任意构建中改进编译次数, 并且降低 I/O 数量. ts 仓库本身也是一个很好的参考, 我们有一些 library 项目和一些 endpoint 项目,  endpoint 项目尽可能的小并且只获取需要的库.

> 翻译自 --https://www.typescriptlang.org/docs/handbook/project-references.html

# 在剩余参数中的元组(Tuples)和扩展表达式

## 在 ts3.0中加入了多个关于与函数参数列表有关新属性作为 tuple 类型

- 元祖类型的剩余参数
```ts
declare function foo(...args: [number, string, boolean]): void;
declare function foo(args_0:number, args_1:string, args_2:boolean): void;
```
- 元组类型的spread表达式: 
```ts
const args: [number, string, boolean] = [42, "hello", true]
foo(42, "hello", true);
foo(args[0], args[1], args[2])
foo(...args)
```
- 普通剩余参数: 可利用该特性抽象出高阶函数
```ts
declare function bind<T, U extends any[], V>(f:(x:T, ...args: U) => V, x:T): (...args: U) => V;

declare function f3(x:number, y:string, z:boolean): void;
const f2 = bind(f3, 42);// (y:string, z:boolean) => void
const f1 = bind(f2, "hello");//(z:boolean) => void
const f0 = bind(f1, true);// () => void
f3(42, "hello", true)
f2("hello", true)
f1(true)
f0()
// T: number, U:[string, boolean], V: void
```
- 元组类型的可选元素: `?`
```ts
let t:[number, string?, boolean?];
t = [42, "hello", true]
t = [42, "hello"]
t = [42]
// 在-- strictNullCheck 模式中, ? 自动回包含`undefined`, 类似于可选参数
```
    - 元组中的剩余元素
```ts
function tuple<T extends any[]>(...args: T): T {
    return args
}
const numbers: number[] = getArrayOfNumbers();
const t1 = tuple('foo', 1, true); // [string, number, boolean]
const t2 = tuple('bar', ...numbers);// [string,...numbers]
```

## 一个新的 未知 (unknown)类型
- unknown 是一个类似于 any 的安全类型
- 任何类型都可以给 unknown 进行赋值
- unknown 不能在没有类型断言或基于窄的控制流给它自己或者 any 之外的类型赋值, 
- 没有任何运算符允许使用 unknown, 在没有初次断言或狭窄定义一个具体的类型, 只有==, 或!==可以使用
- unknown & 任意类型 T 都为 T
- unknown | 任意类型 T 都为 unknown (any 除外 unknown | any = any)
- keyof unknown 为 never 
```ts
type T40 = keyof any;  // string | number | symbol
type T41 = keyof unknown;  // never
```
- 没有属性, 函数调用, 元素索引
```ts
function f11(x: unknown) {
    x.foo;  // Error
    x[5];  // Error
    x();  // Error
    new x();  // Error
}
```
- 用户自断言, 使用 typeof, instanceof
```ts
declare function isFunction(x: unknown): x is Function;

function f00(x: unknown) {
    if (typeof x === "string" || typeof x === "number") {
        x;  // string | number
    }
    if (x instanceof Error) {
        x;  // Error
    }
    if (isFunction(x)) {
        x;  // Function
    }
}
```
- unknown 不能给 object 类型赋值
```ts
function foo(x: { [x: string]: unknown }) {
    x = {};
    x = { a: 5 };
    x = [1, 2, 3];
    x = 123;  // Error
}
```
- unknown 的本地类型被视为初始化
```ts
function foo(x: {}, y: unknown, z: any) {
    let o1 = { a: 42, ...x };  // { a: number }
    let o2 = { a: 42, ...x, ...y };  // unknown
    let o3 = { a: 42, ...x, ...y, ...z };  // any
}
```
- unknown 返回值被视为不需要任何 return 语句
```ts
function foo(): unknown {
}
```
- 剩余类型不能从 unknown 创建
```ts
function foo(x: unknown) {
    let { ...a } = x;  // Error
}
```
- 类属性的未知类型不需要明确的赋值
```ts
class C1 {
    a: string;  // Error
    b: unknown;
    c: any;
}
```