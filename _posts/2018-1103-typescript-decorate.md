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
        c.constructor = constructor.prototype;
        return new c();
    }
    // 添加构造函数行为
    const f:any = function(...args) {
        console.log("new:", origin.name);
        return construct(origin, args)
    }
    // 使得instanceof可用, 链接到原型链上
    f.constructor = origin.prototype;
    return f
}
```
#### 在为类添加额外的逻辑或元数据时, 需要接受一个参数即这个类的构造函数, 其中的target就是类的构造函数, 首先用变量进行保存, 然后对这个构造函数的过程进行修改, 返回一个新的构造函数, 保留原有的原型链. 最终在`new Person('lorry', 'jiang')`的时候会打印`new: lorry`

### 函数装饰器