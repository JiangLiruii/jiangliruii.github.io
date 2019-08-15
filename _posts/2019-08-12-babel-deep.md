---
layout:       post
title:        "深入理解 babel"
subtitle:     "关于 babel 的深入理解"
date:         2019-08-12 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/babel.png'
catalog:      true
multilingual: false
tags:
    - web
---

babel是前端几乎使用量最大的编译工具之一, 他并不像其他语言里的编译器, 把源码编译成机器码, 他更多的是一个转译的功能, 将我们写的代码转换成浏览器可以识别的代码.因为浏览器就是前端的爸爸, 具体怎么转成机器码就是浏览器做的事情了, 比如 v8 引擎.

我在工作中使用 babel 是无感知的, 写代码, 调试代码都没有去看过源代码被编译后的内容, 通过一系列的 presets, plugins的一顿操作, 就写好了, 心血来潮, 想写一篇 babel 如何转原代码的过程. 不过不会包罗全部的语法转换过程, 只写我感兴趣的. 如果你有需要可以自行尝试, 还是蛮好玩的.可以在 [babelIO](https://babeljs.io)练练手.

## class 的实例属性的 shortcut

```js
class Cat{
  run = () => console.log('running')
}
```
其实我现在的版本chrome 76 浏览器已经可以正确解释这段代码了, 但是我们要向后兼容呀. 不兼容到 IE就万事可商量.来看一下 babel 为我们转译后的代码

```js
"use strict";
// 鉴别是否使用new 关键字来实例化的类
function _instanceof(left, right) {
  if (
    right != null &&
    typeof Symbol !== "undefined" &&
    right[Symbol.hasInstance]
  ) {
    return !!right[Symbol.hasInstance](left);
  } else {
    return left instanceof right;
  }
}
// 定义要按照类调用的方法
function _classCallCheck(instance, Constructor) {
  // 如果不是 new 调用, 则报错
  if (!_instanceof(instance, Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}
// 设置属性
function _defineProperty(obj, key, value) {
  // 如果是可枚举
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true
    });
  } else {
    obj[key] = value;
  }
  return obj;
}

var Cat = function Cat() {
  _classCallCheck(this, Cat);

  _defineProperty(this, "run", function() {
    return console.log("running");
  });
};
```

说一下这个`Symbol.hasInstance` 这个构造器对象用于判断一个对象是不是他的实例的方法.也是 `instanceof` 调用的方法, 当然也可以进行自定义

```js
class Cat{
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance)
  }
}

[] instanceof Cat // true
```

## 装饰器
```js
function addName(target) {
}
@addName
class Cat{
  run = () => console.log('running')
}
```

其余部分都一样, 不同的再与 Cat的定义上. 说一个语法表示上的细节, 为更好的阅读 bubble 之后的代码

```js
console.log((1,2,3)) 
```
打印的是什么值呢? 3, ()是表示从左往右依次计算, 直到最右边的执行完毕之后返回.可以将代码变得更加紧凑 

```js
function addName(target) {
}

var Cat =
  addName(
    (_class = ((_temp = function Cat() {
      _classCallCheck(this, Cat);

      _defineProperty(this, "run", function() {
        return console.log("running");
      });
    }),
    _temp))
  ) || _class;
```
可以看出 _temp 就是之前生成的 Cat, 注意这里 addNames 的实现, 调用 addName 一定要返回一个值, 否则就采用原来的类定义.如何写类装饰器也一目了然了.

```js
function addName(target) {
  // 这里实质上修改的是静态属性
  target.name = 'beauty'
  // 为了代码可读性, 最好返回, 不返回也没太大关系, 因为 target 为引用对象, 也可以修改成功
  return target
}
```

下面再看看类的实例属性装饰器

```js
function readOnly(target, key, descriptor) {
  descriptor.writable = false
  return descriptor
}

class Cat{
  @readOnly
  run () {console.log('running');}
}
```

这里的改动就比较大了, 我把整个贴出来, 且在线演示版本无法同时使用简写版类属性与装饰器, 所以这里用的正常的类实例方法定义.
```js

"use strict";

var _class, _descriptor, _temp;

function _instanceof(left, right) {
  // same
}

function _classCallCheck(instance, Constructor) {
  // same
}
// 同时设置class 的多个属性
function _defineProperties(target, props) {
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    descriptor.enumerable = descriptor.enumerable || false;
    descriptor.configurable = true;
    if ("value" in descriptor) descriptor.writable = true;
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}
// 
function _createClass(Constructor, protoProps, staticProps) {
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  if (staticProps) _defineProperties(Constructor, staticProps);
  return Constructor;
}

/**
 *  应用装饰器描述符
 *  target: 类的原型, property: 属性名称, decorators: 装饰器列表, descriptor: 属性的描述符, context: 指定的上下文
 */
function _applyDecoratedDescriptor(
  target,
  property,
  decorators,
  descriptor,
  context
) {
  var desc = {};
  // 遍历应用每一个描述符, 可枚举的属性都会包括 enumerable, configurable, value, writable, 跟 Object.defineProperty 传入的属性一样.
  Object.keys(descriptor).forEach(function(key) {
    desc[key] = descriptor[key];
  });
  desc.enumerable = !!desc.enumerable;
  desc.configurable = !!desc.configurable;
  // 当有 value 为值属性的时候, 才设置 writable
  if ("value" in desc || desc.initializer) {
    desc.writable = true;
  }
  // 将 decorators 的顺序反向后(所以第一个添加的修饰器最后一个处理)
  desc = decorators
    .slice()
    .reverse()
    .reduce(function(desc, decorator) {
      return decorator(target, property, desc) || desc; // 装饰器函数一定要返回一个 descriptor, 否则采用原有的 desc 对象, 当然, 因为是对象, 一个引用, 不返回也能修改该对象的值.
    }, desc);
  /**
   * 这里为什么要用 void 0 ? void 0 完全等同于 undefined, 只是 undefined 在 js 中不是关键字, 可能被重定义, void 0 比 undefined 更严谨
   * 也可以使用 void 'hello world', void 1000000 等, 都是 undefined, 为什么不用更简单的 void 0 表示呢
   * initializer 可以在装饰器中指定修改
   */
  if (context && desc.initializer !== void 0) {
    desc.value = desc.initializer ? desc.initializer.call(context) : void 0; // 调用初始化方法, 生成值
    desc.initializer = undefined;
  }
  if (desc.initializer === void 0) {
    Object.defineProperty(target, property, desc);
    desc = null;
  }
  return desc;
}
function readOnly(target, key, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

var Cat = ((_class =
  /*#__PURE__*/
  (function() {
    // 将 Cat 作为一个闭包传入, 防止与外部的变量冲突
    function Cat() {
      _classCallCheck(this, Cat);
    }

    _createClass(Cat, [
      {
        key: "run",
        value: function run() {
          console.log("running");
        }
      }
    ]);

    return Cat;
  })()),
_applyDecoratedDescriptor(
  _class.prototype,
  "run",
  [readOnly],
  Object.getOwnPropertyDescriptor(_class.prototype, "run"),
  _class.prototype
),
_class);
```

可以尝试修改实例的 run 方法
```js
var c = new Cat // Cat的构造函数中没有参数, 可以省略()

c.run = 1;// use strict 环境下会报错
c.run() // running, 没有被修改
```
多增加一个装饰器
```js
function readOnly(target, key, descriptor) {
  descriptor.writable = false
  return descriptor
}
function initialize(target, key, descriptor) {
  descriptor.writable = true;
  descriptor.initializer = () => 1000
  // 注意此函数没有任何返回, 且使用了 initializer 属性, 该 key 的 value 值为 initializer()执行后的结果
}

class Cat{
  // 先应用 initialize, 再应用 readOnly, 可以通过Object.getOwnPropertyDescriptor(Cat.prototype, 'run')检测 writable 属性证明
  @readOnly@initialize
  run () {console.log('running');}
}

const c = new Cat
c.run;// 1000, 这时候就不是方法了, 完全重写了原有的 console 的定义.
```

看完了装饰类的, 再看看装饰对象的有什么不同

```js
function initialize(target, key, descriptor) {
  descriptor.writable = true;
  descriptor.initializer = () => 1000
}
const a = {
  @initialize
	name: 'lorry'
}

a.name; // 1000
```

转换后的代码, 核心还是`_applyDecoratedDescriptor`这个函数.

```js
var a = ((_obj = {
  name: "lorry"
}),
_applyDecoratedDescriptor(
  _obj,
  "name",
  [initialize],
  ((_init = Object.getOwnPropertyDescriptor(_obj, "name")),
  (_init = _init ? _init.value : undefined),
  // 将 descriptor 全部重新定义, 要定义 enumerable 和 configurable, writable 的值的话需要使用 Object.defineProperty()是不可能有添加装饰器的机会的
  {
    enumerable: true,
    configurable: true,
    writable: true,
    //  如果是对象会自带一个 initializer, 与类最不同的地方没有 value 属性了.
    initializer: function initializer() {
      return _init;
    }
  }),
  _obj
),
_obj);
```

## 私有变量

chrome 在 74 版本的时候就支持类的私有属性了, 比如

```js
class Beauty {
  #age = 18;
  getAge() {
    return this.#age
  }
}

const beautyGirl = new Beauty;
beautyGirl.getAge() // 18
beautyGirl.#age // Error
```
该私有属性除了 Beauty 类本身可以获取到之外, 其余没有地方可以获取.包括继承其的类.

那么 babel 会转换成怎么样可识别的代码呢?

```js
// 获取私有字段方法, 必须通过该方法才可获取到私有字段
function _classPrivateFieldGet(receiver, privateMap) {
  var descriptor = privateMap.get(receiver);
  if (!descriptor) {
    // 只能在实例中去获取该属性, 即 class 中的 this
    throw new TypeError("attempted to get private field on non-instance");
  }
  // 如果是通过 getter 方式设置的
  if (descriptor.get) {
    return descriptor.get.call(receiver);
  }
  // 否则返回值
  return descriptor.value;
}

class Beauty {
  constructor() {
    _age.set(this, {
      writable: true,
      value: 18
    });
  }

  getAge() {
    return _classPrivateFieldGet(this, _age);
  }
}
// 使用 WeakMap 结构, 每一个私有变量都会 new 一个 WeakMap
var _age = new WeakMap();
```

最关键的是看出其使用的 WeakMap 结构, 跟普通字典{}的区别在于, WeakMap 必须以对象作为键. (因为其弱引用的机制)那么 WeakMap 与 Map 有什么区别呢? **垃圾回收**, WeakMap 的键一旦被 delete 调了之后是不会计算其内部的引用而被垃圾回收调, Map 的话则会继续保留引用, 从而不会被垃圾回收. 也正是由于这种弱引用的机制, WeakMap 里面的键和值都是不可枚举的.

```js
var weakM = new WeakMap()
var obj = {value: 1}
weakM.set(obj, {writable: false, value: obj.value})
window.obj = obj
delete window.obj// 这个时候 obj 就已经做好被垃圾回收的准备了, 即便 set 里面还保留着对 obj 的引用
```

再来看看如何进入`_classPrivateFieldGet`方法的以下条件判断
```js
  if (descriptor.get) {
    return descriptor.get.call(receiver);
  }
```

需要设置私有变量为 getter, 也就转变为私有方法了

```js
class Beauty {
  get #age() {
    return 18
  }
  setAge(age) {
    this.#age = age
  }
  printAge() {
    console.log(this.#age)
  }
}

// 转换为
// 获取私有方法的 polyfill
function _classPrivateMethodGet(receiver, privateSet, fn) {
  // 判断是否将该实例放到了私有 set 中
  if (!privateSet.has(receiver)) {
    throw new TypeError("attempted to get private field on non-instance");
  }
  return fn;
}
// 注意此处的设置私有方法时的报错

function _classPrivateMethodSet() {
  throw new TypeError("attempted to reassign private method");
}

class Beauty {
  constructor() {
    _age.add(this);
  }

  setAge(age) {
    _classPrivateMethodSet();
  }

  printAge() {
    console.log(_classPrivateMethodGet(this, _age, _age2));
  }
}
// 储存私有变量的数据, 每一个私有变量都会 new 一个 WeakSet, 注意与私有变量 new 的 WeakMap 不同, 这里 new 的是 WeakSet,  Set 和 Map 的区别在于 key, value是否相同, 或者说 set只有  key 没有 value(或反之), 在私有方法中, value 的值是同名函数调用得到, 所以只储存一个即可.
var _age = new WeakSet();

var _age2 = function _age2() {
  return 18;
};

```

需要注意的是, 上例可以看出私有变量的 get 或者为method 的是无法进行修改的.但是值是可以的, 比如

```js
class Beauty {
  #age = 18
  setAge(age) {
    this.#age = age
  }
}
// 转换为

// 设置类的私有字段
function _classPrivateFieldSet(receiver, privateMap, value) {
  // 获取字段自描述对象
  var descriptor = privateMap.get(receiver);
  if (!descriptor) {
    throw new TypeError("attempted to set private field on non-instance");
  }
  if (descriptor.set) {
    // 调用 WeakMap 的 set 方法
    descriptor.set.call(receiver, value);
  } else { // 如果没有 set 方法, 即不是 WeakMap
    // 如果可写的话
    if (!descriptor.writable) {
      throw new TypeError("attempted to set read only private field");
    }
    // 直接设置值
    descriptor.value = value;
  }
  return value;
}

class Beauty {
  constructor() {
    _age.set(this, {
      writable: true,
      value: 18
    });
  }

  setAge(age) {
    _classPrivateFieldSet(this, _age, age);
  }
}

var _age = new WeakMap();

```

注意这里的 `_classPrivateFieldSet` 方法.

相信之后 babel 还会再进行优化, 现在私有变量还有诸多的限制, 以后应该会有更大的发挥空间, 可以像 Java 那样有 protect 可被继承类访问到的受保护的变量.

## async / await

再看一个有趣的, 异步的语法.工作中用得蛮多的.

```js
function a () {
  return new Promise(resolve => resolve(true))
}

async function b() {
  const boolValue = await a()
}
```

```js
"use strict";
// 异步生成器步骤, 需要传入的参数为: 生成器本身, promise 的 resolve 和 reject, _next 和_throw 状态控制方法的引用, key 状态控制方法名 arg next 对应的值, throw 对应的 err
function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
  try {
    var info = gen[key](arg);
    var value = info.value;
  } catch (error) {
    reject(error);
    return;
  }
  if (info.done) {
    resolve(value);
  } else {
    Promise.resolve(value).then(_next, _throw);
  }
}
// 异步生成器
function _asyncToGenerator(fn) {
  return function() {
    var self = this,
      args = arguments;
    return new Promise(function(resolve, reject) {
      var gen = fn.apply(self, args);
      function _next(value) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value);
      }
      function _throw(err) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err);
      }
      _next(undefined);
    });
  };
}

function a() {
  return new Promise(function(resolve) {
    return resolve(true);
  });
}
// 转换之后的 b 函数, 保留this上下文和 arguments
function b() {
  return _b.apply(this, arguments);
}

function _b() {
  _b = _asyncToGenerator(
    /*#__PURE__*/
    // 标明为 generatorFunction, 并连接原型链到 generator prototype 上
    regeneratorRuntime.mark(function _callee() {
      // await后的变量
      var boolValue;
      return regeneratorRuntime.wrap(function _callee$(_context) {
        while (1) {
          // 更新_context的状态
          switch ((_context.prev = _context.next)) {
            case 0:
              // 开始等待状态
              _context.next = 2;
              // 返回一个 promise, 开始等待 promise resolve 或 rejectr
              return a();
              //  获取到 resolve 的值赋给 boolValue
            case 2:
              boolValue = _context.sent;
            // end 时终止yield
            case 3:
            case "end":
              return _context.stop();
          }
        }
      }, _callee);
    })
  );
  return _b.apply(this, arguments);
}
```

上述代码无法直接贴到浏览器中正常运行, 因为里面有一个 regeneratorRuntime, 他是生成器的一个运行时,如果要解释这个, 可以再写另外一篇文章了, 有兴趣的可以先看看[这里](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js).简单来说他就是一个状态机, 维护 4 中状态:
```js
  var GenStateSuspendedStart = "suspendedStart";
  var GenStateExecuting = "executing";
  var GenStateSuspendedYield = "suspendedYield";
  var GenStateCompleted = "completed";
```

async 和 await 的语法还没研究透, 上述只是很浅显的理解, 还需要对源码去深入了解状态是如何转换的.一时半会没看懂, 看懂了再来更新.

先写到这, 之后遇到再补充