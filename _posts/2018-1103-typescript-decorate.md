---
layout:       post
title:        "typescript decorator"
subtitle:     "typescript 的装饰器"
date:         2018-11-03 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/performanceEvent.jpg'
catalog:      true
multilingual: false
tags:
    - web
---

# 注解和装饰器都是从ts1.5以后就有的新特性, 主要包括:

- 类装饰器
- 方法(函数)装饰器
- 属性装饰器
- 参数装饰器
- 装饰器工厂
- 带有参数的装饰器

## 注解是一种作为类声明添加元数据的方法, 元数据就可以被注入依赖注入容器这样的工具所使用. 注解最早是由Google 的 AtScript提出, 但最终没有成为语言标准, 但是装饰器由 Yehuda Katz提出, 已是ECMAScript 7 标准的特性.

## 装饰器跟注解很像同一个东西, 使用上来看具有相同的语法, 但是使用注解时不需要关心如何将元数据加入代码里来的, 装饰器更像是一个接口, 用来构建一个以注解位结尾的东西

### 我们将使用下面这个类来说明如何使用装饰器
```ts
class Person {
    public name: string;
    public surname: string;

    constructor (name:string, surname:string) {
        this.name = name;
        this.surname = surname;
    }

    public saySomething(something:string):string {
        return this.name +' ' + this.surname + ' says: '+ something; 
    }
}
```
### 类装饰器

#### 类装饰器用来修改类的构造函数, 如果装饰器函数返回undefined, 那么类仍然使用原来的构造函数, 如果装饰器由返回值, 那么返回值会覆盖原来的构造函数,下面创建一个logClass的装饰器
```ts
function logClass(target: any) {

}
@logClass
class Person {
    public name:string
    //...
}
```
#### 使用gulp和tsc编码之后, 会有一个__decorate的代码实现, 内部实现先按下不表, 看一下转译后的类
```js
function logClass(target) {
}
var Person = /** @class */ (function () {
    function Person(name, surname) {
        this.name = name;
        this.surname = surname;
    }
    Person.prototype.saySomething = function (something) {
        return this.name + ' ' + this.surname + ' says: ' + something;
    };
    Person = __decorate([
        logClass
    ], Person);
    return Person;
}());
```
#### 知道了代码, 下面看看logClass的逻辑实现
```js
function logClass(target:any) {
    const  origin = target;
    // 用来生成类的实例的工具方法
    function construct(constructor, args) {
        const c:any = function() {
            return constructor.apply(this, args);
        }
        c.prototype = constructor.prototype;
        return new c();
    }
    // 添加构造函数行为
    const f:any = function(...args) {
        console.log("new:", origin.name);
        return construct(origin, args)
    }
    // 使得instanceof可用, 链接到原型链上
    f.prototype = origin.prototype;
    return f
}
```
#### 在为类添加额外的逻辑或元数据时, 需要接受一个参数即这个类的构造函数, 其中的target就是类的构造函数, 首先用变量进行保存, 然后对这个构造函数的过程进行修改, 返回一个新的构造函数, 以原有构造函数的参数为参数, 保留原有的原型链. 最终在`new Person('lorry', 'jiang')`的时候会打印`new: lorry`

### 函数装饰器

#### 函数装饰器也成为方法装饰器, 它包含一个接受三个参数的函数, 这三个参数分别是:

- 这个属性的对象
- 属性名(一个字符串或一个符号)
- 可选参数(属性的描述对象)

#### 这个函数会返回一个undefined或参数里提供的属性的描述对象, 或这一个新的描述对象(即覆盖掉原方法). 返回undefined等同于返回参数提供的描述对象
> 属性的描述对象可以通过 Object.getOwnPropertyDescriptor()方法获取对象

#### 先定义一个没有任何行为的方法
```ts
function logMethod(target:any, key:string, descriptor:any) {
    // ...
}
// 用于装饰Person类
class Person {
    @logMethod
    public saySomthing(something:string) {

    }
}
// 转译后的Person类
var Person = /** @class */ (function () {
    function Person(name, surname) {
        this.name = name;
        this.surname = surname;
    }
    Person.prototype.saySomething = function (something) {
        return this.name + ' ' + this.surname + ' says: ' + something;
    };
    __decorate([
        logMethod
    ], Person.prototype, "saySomething", null);
    Person = __decorate([
        logClass
    ], Person);
    return Person;
}())
```
#### 该装饰器被调用时, 带有以下参数
- 包含了被装饰方法的类的原型: Person.prototype
- 被装饰方法的名字: saySomething
- 被装饰方法的属性描述对象: Object, 一般为 `{value: ƒ, writable: true, enumerable: true, configurable: true}`

```ts
function logMethod(target:any, key:string, desctiptor:any) {
    // 保留原方法的引用
    var originMethod = descriptor.value
    // 编辑descriptor中的value属性, args为方法对应的参数
    descriptor.value = function(...args:any[]) {
        // 将方法参数转化为字符串
        var a = args.map(a => JSON.stringify(a)).join();
        // 执行方法, 得到原返回值
        var result = originMethod.apply(this, args)
        // 将返回值转为字符串
        var r = JSON.stringify(result);
        console.log(`call: ${key}(${a}) => ${r}`);
        // 返回方法调用结果
        return result
    }
    // 返回编辑后的descriptor对象
    return descriptor;
}
```
#### 跟实现类装饰器一样, 创建被装饰元素的副本开始, 没有直接调用target[key]来访问, 而是通过属性描述对象 (descriptor.value), 然后创建一个新函数来替代被修饰函数, 新函数除了调用原函数之外, 还包含了额外逻辑.
` saySomething("world") => "Remo jansen says: world"
"Remo jansen says: world" `