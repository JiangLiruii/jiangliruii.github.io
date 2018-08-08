---
layout:       post
title:        "js中的比较运算"
subtitle:     "常用的比较都不会出错, 就怕有了非常见的比较却不知道导致的bug"
date:         2018-07-31 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/compare.jpg'
catalog:      true
multilingual: false
tags:
    - javascript
---

ECMAScript标准中有一大章节是讲[Abstract Operation](http://www.ecma-international.org/ecma-262/8.0/index.html#sec-abstract-operations), 目的仅仅是为了定义语法以及抽象运算, 其中就包括了比较,以及比较中十分重要的类型转换.

## 类型的转换

都知道js不是显示类型的语言.比如

```js
var a = 1;
a = '123';
```
a既可以是数字也可以是字符串, 并不会报错,在其他强类型语言中可不能如此随意.

### ToPrimitive(input[,PreferredType])
转换流程
- 首先判断是否是ECMASctipt语言类型
    - Undefined
    - Null
    - Boolean
    - String
    - Number
    - Symbol
    - Object
- 如果是Object, 则比对PreferredType的值
    - 没有传 则 hint -> default
    - String 则 hint -> string
    - Number 则 hint-> number
    - 新建一个抽象概念为: exoticToPrim, 它表示GetMethod(input, @@toPrimitive)
    - 如果exoticToPrim不为undefined, 则
        - 新建一个result为Call(exoticToPrim, input, << hint >>)
        - 如果result不为Object, 则返回result
        - Throw TypeError
    - 如果hint是default 设置hint为number
    - 返回 OrdinaryToPrimitive(input, hint)
- 返回input

> GetMethod(V, P) V --> V 为基本类型, P是属性key 执行流程为
> IsPropertyKey(P) 如果P是String或Symbol返回true, 否则返回false 
> 使 func = GetV(V, P) 相当于 P.V

> Call(F, V[, arguments]) 1 如果arguments没有传,则为空list 2 如果F不能调用,则TypeError, 3 返回F(V, arguments)

> OrdinaryToPrimitive(O, hint) 
> 1 判定O是否为字典 
> 2 判定 hint是String, 并且它的值为string 或 number 
> 3 如果hint是string, 让methodName 为 << 'toString', 'valueOf' >>,否则为<< 'valueOf', 'toString' >> 
> 4 对methodName中进行遍历, 让method = O.name, 如果method可以被调用,那么result为method().call(O), 如果result不是字典,返回result 
> 5 TypeError

* 注意: @@toPrimitive 如果在调用时没有传入hint, 则通常会将hint视为Number, 但是对象可以重写该行为, 但是仅有Date object 和 Symbol object可以重刻默认的ToPrimitive, Date对象将会把没有hint视为String的hint

#### 是不是看了不够明白什么意思? 没关系,接着往下看,你会看到他的用武之地

### ToBoolean(argument)

- undefined --> false
- Null --> false
- Boolean --> argument
- Number --> +0, -0, NaN返回false, 其余返回true
- String --> 如果字符串为空 '', 返回false, 其余返回true
- Symbol --> 返回true
- Object --> 返回true

### ToNumber(argument) 

- Undefined --> NaN
- Null --> +0
- Boolean --> true返回1, false返回0
- Number --> 返回argument
- String --> 参考下述
- Symbol --> TypeError
- Object --> 递归返回ToNumber(ToPrimitive(argument, hint Number)) **这里用了ToPrimitive 以及hint参数**

### ToInteger(argument)

- number = ToNumber(argument)
- 如果number 为NaN 返回 +0
- 如果number为 +0, -0, +infinity, -infinity, 返回number
- 返回 floor(abs(number))

### ToInt32(argument) 将argument 转化为one of 2<sup>32</sup> integer values in the range -2<sup>31</sup> through 2<sup>31</sup>-1, inclusive

- number = ToNumber(argument)
- if number为 +0, -0, +infinity, -infinity, 返回+0
- int = floor(abs(number))
- int32bit = int 对 2**32 求余
- 如果int32bit >= 2<sup>31</sup>, 返回int32bit - 2<sup>32</sup>, 否则返回int32bit

### ToUnit32(argument) 将argument 转化为one of 2<sup>32</sup> integer values in the range 0 through 2<sup>32</sup>-1

- 前4步都一样,唯独最后一步, 不需要判断, 直接返回int32bit

### ToInt16(argument), ToInt8(argument)将ToInt32中2<sup>32</sup>换成2<sup>16</sup>, 2<sup>8</sup>

### ToUnit16(argument), ToUnit8(argument)将ToUnit32中2<sup>32</sup>换成2<sup>16</sup>, 2<sup>8</sup>

### ToUnit8Clamp(argument)

- number = ToNumber(argument)
- if number = NaN, 返回+0
- if number<=0, 返回+0
- if number >=255 返回255
- f = floor(number)
- if f + 0.5 < number, 返回 f + 1 否则返回f
- if f 是奇数, 返回f+1
- 返回 f

### ToString(argument)

- Undefined --> "Undefined"
- Null --> "Null"
- Boolean --> true -> "true" false -> "false"
- Number --> NaN -> "NaN" +0,-0 -> '0'  小于0 -> "-" + !ToString(-number) +∞ -> "Infinity" 其余的返回对应数字
- String --> 返回argument
- Symbol --> TypeError
- Object --> primValue = ToPrimitive(argument, hint String) return ToString(primValue)

### ToObject(argument)

- Undefined, Null --> TypeError
- Boolean, Number, String, Symbol --> 返回对应的一封装的object(new String())
- Object --> argument

说了那么多抽象方法,下面来看看如何把这些抽象方法用起来

### Abstract Relational Comparison, 抽象比较方法, 即大小于的实现原理
例如 x < y
- px = ToPrimitive(x), py = ToPrimitive(y) (LeftFirst = true)
- 如果px和py都是字符串
    - py是px的前缀, 返回false
    - px是py的前缀, 返回true
    - k为最小不同位索引, 比如 'hello' 和 'ha' k = 1
    - m = px[k], n = py[k]
    - if m < n 返回true, 否则返回false
- 如果不是字符串
    - nx = ToNumber(px), ny = ToNumber(py)
    - if nx, ny任一 为NaN, 返回undefined,
    - 如果nx和ny有相同的Number value, 返回false
    - 如果nx是+0, ny是-0 返回false
    - 如果nx是-0, ny是+0, 返回false
    - 如果nx是+∞, 返回false
    - 如果ny是+∞, 返回true
    - 如果ny是-∞, 返回false
    - 如果nx是-∞, 返回true
    - 数字比较nx ny

### Abstract Equality Comparison 抽象相等比较

x == y

- 如果x和y是相同的类型, 则返回 x === y
- 如果x为null y为undefined, 返回true
- 如果x为undefined y为null, 返回true
- 如果x为数字, y为String, 返回 x == ToNumber(y)
- 如果x是String, y为Number, 返回 ToNumber(x) == y
- 如果x是Boolean, 返回ToNumber(x) == y
- 如果y是Boolean, 返回x == ToNumber(y)
- 如果x不为String, Number或Symbol, y是Object, 返回x == ToPrimitive(y)
- 如果x是Object, y不是String, Number 或 Symbol, 返回 ToPrimitive(x) == y
- 返回false

### 严格相等比较

x === y

- 如果x,y类型不同, 返回false
- 如果x是Number, 则
    - x, y任一是NaN, 返回false
    - x 跟y有相同Number值, 返回true
    - x 是+0,-0 y是-0,+0,返回true
    - 返回false
- 返回SameValueNonNumber(x,y)

#### SameValueNonNumber(x,y)
- x不为数字, 且x与y类型相同
- 如果x是undefined或null, 返回true
- 如果x是字符串, x和y敲好有相同的length, code units, 染回true, 否则返回false
- 如果x是boolean, 如果x和y同时为true或false, 返回true,否则为false
- x为symbol, 如果x和y有同样的symbol值, 返回true, 否则返回false
- 如果x,y有相同的Object value, 返回true, 否则返回false.


    