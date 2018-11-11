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

### 属性装饰器
#### 包含两个参数: 1 这个属性的对象 2 这个属性的属性名(字符串或符号) 不会返回一个属性的描述对象.
```ts
function logProperty(target:any, key:string) {
    // ...
}
class Person {
    @logProperty
    public name: string;
    //...
}
```
#### 我们将使用一个新属性来替代原来的属性, 新属性会表现的和原属性一致, 除了添加的打印的功能

```ts
function logProperty(target:any, key:string) {
    var _val = this[key]
    var getter = function() {
        console.log(`Get: ${key} => ${_val}`)
    }
    var setter = function (newVal) {
        console.log(`Set: ${key} => ${newVal}`)
        _val = newVal;
    }
    // 删除属性在严格模式下如果对象configurable为false时是不可用的, 会报错.非严格模式会返回false
    if (delete this[key]) {
        Object.defineProperty(target, key, {
            get: getter,
            set: setter,
            enumerable: true,
            configurable: true,
        })
    }
}
```

#### 在使用了上述监视器时候就可以在每次设置或获取属性时在控制台观察每一次的变化.

### 参数装饰器

#### 接受三个参数, 1 包含被装饰参数的方法的对象 2 方法的名字(或undefined) 3 参数在参数列表中的索引, 该装饰器的返回值会被忽略

```ts
function addMetaData(target:any, key:string, index:number) {
    // ...
}

public saySomething(@addMetaData something:string) : string {
    return this.name + ' ' + this.surname + ' says: ' + something;
}
```
#### 参数属性没有返回值, 意味着不能对包含被修饰参数的方法进行覆盖
```ts
function addMetaData(target:any, key:string, index:number) {
    var metadataKey = `_log_${key}_parameters`;
    if (Array.isArray(target[metadataKey])) {
        target[metdataKey].push(index);
    } else {
        target[metadataKey] = [indedx]
    }
}
```
#### 为了让更多的参数可以被装饰, 将检查这个新属性是否为一个数组. 单独的参数装饰器不是很有用, 但是联合方法装饰器, 就可以将参数装饰器所添加的元数据读取出来

```ts
@readMetadata
public saySomething(@addMetaData something:string) : string {
    return this.name + ' ' + this.surname + ' says: ' + something;
}
// 仅仅打印被装饰的参数
function readMetadata(target:any, key:string, descriptor: any) {
    var origin = descriptor.value;
    descriptor.value = function (...args: any[]) {
        var metadataKey = `_log_${key}_parameters`;
        var indices = target[metadataKey];
        if (Array.isArray(indices)) {
            for (var i = 0; i < args.length; i++) {
                if (indices.indexOf(i)) {
                    var arg = args[i];
                    var argStr = JSON.stringify(arg) || arg.toString();
                    console.log(`${key} args[${i}]: ${argStr}`)
                }
            }
            var result = origin.apply(this, args)
            return result;
        }
    }
    return descriptor;
}
```
#### 由此, 当new 一个Person的实例后调用saySomething 的方法时便会展示添加的元数据. 但是需要注意的是这里使用了类属性的数组target[metadataKey]来保存数组, 下面来介绍使用反射元数据API, 用于专门生成和读取元数据

### 装饰器工厂

#### 一个接受任意数量参数的函数, 并且返回上述的任意一种装饰器

#### 上述已经写了如何实现一个装饰器, 但是大多数情况下都是去使用而不是去实现, 可以使用装饰器工厂来使装饰器更容易被使用
```ts
@logClass
class Person {
    @logProperty
    public name:string;
    public surname:string;

    constructor(name:sting, surname:stirng){
        this.name = name
        this.surname = surname
    }
    @logMethod
    public saySomething (@logParameter something:string) : string{
        return this.name +' ' + this.surname + ' says: '+ something; 
    }
}
// 使用工厂装饰器
@log
class Person {
    @log
    public name:string;
    public surname:string;

    constructor(name:sting, surname:stirng){
        this.name = name
        this.surname = surname
    }
    @log
    public saySomething (@log something:string) : string{
        return this.name +' ' + this.surname + ' says: '+ something; 
    }
}
```
#### 这样就可以使用一个log的装饰器, 而无需担心是否使用了正确的类型. 装饰器工厂能够鉴别该使用哪种装饰器并返回函数
```ts
function log(...args:any[]) {
    switch (args.length) {
        case 1:
            return logClass.apply(this, args)
        case 2:
            // 属性装饰器没有返回值, 所以使用break替代return
            logProperty.apply(this, args)
            break
        case 3:
            // 参数装饰器第三个参数为index:number, 且参数装饰器要结合函数装饰器使用
            if(typeof args[2] === 'number') {
                logParameter.apply(this, args)
            }
            return logMethod.apply(this, args);
        default:
            throw new Error('Decorators are not valid here')
    }
}
```
#### 正如我们看到的, 装饰器工厂通过参数的数量和类型来判断返回的装饰器类型

## 带有参数的装饰器
### 可以使用一种特殊的装饰器工厂来配置装饰器的行为
```ts
@logClass('option')
class Person {
    //...
}
```
#### 为了给装饰器传递参数, 需要使用一个函数来包裹装饰器, 这个包裹函数接受参数并返回一个装饰器.即一个高阶函数
```ts
function logClass(option:any) {
    return function(target:any) {
        // 类装饰器就可以访问到参数的内容了
        console.log(target, option)
    }
}
```
#### 可以运用到之前讨论的任意一种装饰器中

## 反射元数据API
### 现在仅仅在代码设计时使用类型但实际上更多时候是动态的, 比如依赖注入, 运行时类型断言, 反射和测试, 这些都需要运行时的类型信息.装饰器的作用即是生成元数据, 这些元数据会携带类型信息, 可以在运行时被处理.
### 当一个元素被保留装饰器装饰后, 编译器会自动的添加类型信息到元素上, 这些保留的装饰器如下:

- @type: 被装饰目标序列化后的类型
- @returnType: 被装饰目标若为函数, 则为其序列化后的返回类型, 否则为undefined
- @parameterType: 如果它是一个函数, undefined或其他类型, 讁时被装饰目标序列化后的参数类型
- @name: 被装饰目标的名字

### 最终产生了反射元API ES7的特性之一.

- 类型元数据: design:type
- 参数类型元数据: design:parameter
- 返回值元数据: design:returnType

### ECMAScript的相关介绍
#### 在对象或属性上定义元数据
```ts
Reflect.defineMetadata(metadataKey, metadataValue, target)
Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey)

// 检查在原型链上对象或属性的元数据key
let result = Reflect.hasMetadata(metadataKey, target)
let result = Reflect.hasMetadata(metadataKey, target, propertyKey)

//在原型链上获取对象或属性元数据key对应的的value值
let result = Reflect.getMetadata(metadataKey, target)
let result = Reflect.getMetadata(metadataKey, target, propertyKey)

// 获取自有的元数据(own metadata)
let result = Reflect.getOwnMetadata(metadataKey, target)
let result = Reflect.getOwnMetadata(metadataKey, target, propertyKey)

// 获取在类或属性上所有的元数据key
let result = Reflect.getMetadataKeys(target)
let result = Reflect.getMetadataKeys(target, propertyKey)

// 获取所有自由元数据的keys
let result = Reflect.getOwnMetadataKeys(target)
let result = Reflect.getOwnMetadataKeys(target, propertyKeys)

// 删除元数据
let result = Reflect.deleteMetadata(metadataKey, target)
let result = Reflect.deleteMetadata(metadataKey, target, propertyKey)

// 通过在构造函数上的装饰器应用元数据
class C {
    // 通过装饰方法(属性)来应用元数据
    @Reflect.metadata(metadataKey, metadataValue)
    method() {
        // ...
    }
}

// 预先设计不同的装饰器
function Type(type) {return Reflect.metadata('design:type', type);}
function ParamTypes(...types) { return Reflect.metadata('design:paramtypes', types);}
function ReturnType(type) { return Reflect.metadata('design:returntype', type);}

// 装饰器的应用
@ParamTypes(String, Number)
class C {
    constructor(text, i){}
    @Type(String)
    get name() { return 'text'; }
    @Type(Function)
    @ParamTypes(Number, Number)
    @ReturnType(Number)
    public add(x, y) {
        return x + y
    }
}

// 元数据的自省
let obj = new C('a', 1)
let paramTypes = Reflect.getMetadata('design:paramtypes', obj, 'add') // [Number, Number]
```

### 抽象运算
#### 在对象上的运算
**获取或得到元数据表(GetOrCreatMetadataMap)(O, P, Create)**

> 这里有一个internal slot的引用(内部槽位), 具体可参见: [internal slot](https://stackoverflow.com/questions/33075262/what-is-an-internal-slot-of-an-object-in-javascript)

当抽象运算 GetOrCreatMetadataMap被调用时, 参数分别为 对象 O, 属性的key P, 以及布尔类型的Create, 下列步骤会一次发生
- 断言: P时undefined 或者 IsPropertyKey(P) 是为true 
- 使targetMetadata成为O\[\[metadata\]\]内部slot
- 如果targetMetadata是undefined, 然后
    - 如果create为false, 返回undefined
    - 设置targetMetadata成为新创建的Map对象
    - 设置O的Metadata这个内部slot为targetValue
- 使metadataMap成为targetMetadata[P]
- 如果metadataMap是undefined
    - 如果create为false, 返回undefined
    - 设置metadataMap为新创建的Map对象
    - targetMetadata[P] = metadataMap
- 返回metadataMap
  
### 普通对象(Object)与特殊对象(String, Array等)的行为
#### 普通对象的内部方法和槽位
所有普通对象都有一个被称为Metadata的内部slot, 这个值是null或者Map对象, 并且被用来储存一个对象的元数据
##### metadata存在的检查
当HasMetadata这个O的内部方法被调用的时候, 参数分别为基本类型的MetadataKey和属性key P, 以下步骤会发生
- 返回OrdinaryHasMetadata(MetadataKey. O, P)
来看看这个OrdinaryHasMetadata的含义
当抽象函数被调用时, 发生了下列步骤:
- 断言: P时undefined, 或者IsPropertyKey(P)是true
- 使hasOwn为OrdinaryHasOwnMetadata(MetadataKey, O, P)
- 如果hasOwn为true, 返回true
- 让parent为 O.GetPrototypeOf(),即往原型链上找
- 如果parent不为null, 递归重复parent.HasMetadata(MetadataKey, P)
- 返回false
##### HasOwnMetadata(MetadataKey, P)方法
当O的HasWonMetadata 内部方法被调用的时候, 参数为基本类型MetadataKey和属性keyP, 下面的步骤将会发生
- 返回OrdinaryHashOwnMetadata(MetadataKey, O, P)
    - 断言P是undefined或IsProperty(P)为true
    - 使metadataMap为GetOrCreateMetadataMap(O, P, false)
    - 如果metadataMap是undefined, 返回false
    - 返回ToBoolean(invoke(metadataMap, 'has', MetadataKey))即返回metadataMap中是否含有MetadataKey

##### GetMetadata(MetadataKey, P)
当GetMetadata这个内部方法被调用的时候, 参数还是MetadataKey和P
- 发挥OrdinarygetMetadata(MetadataKey, O, P)
    - 依照惯例断言P
    - 使hasOwn为OrdinaryHasOwnMetadata(MetaddataKey, O, P)
    - 如果hasOwn为真, 返回OrdinaryGetOwnMetadata(MetadataKey, O, P)
    - 让parent为O.GetPrototypeOf
    - 如果parent不为null, 则一直往上找
    - 返回undefined

##### GetOwnMetadata(MetadataKey, P, ParamIndex)
- 返回OrdinaryGetOwnMetadata(MetadataKey, O, P)
    - 按照惯例断言P
    - 使metadataMap为GetOrCreateMetadataMap(O, P, false)
    - 如果metadataMap为undefined, 返回undefined
    - 返回metadataMap\[MetadataKey\]

#### DefineOwnMetadata(MetadataKey, MetadataValue, P)
- 返回OrdinaryDefineOwnMetadata(MetadataKey, MetadataValue, O, P)
    - 按照惯例判断P
    - 使metadataMap为GetOrCreateMetadataMap(O, P, true)
    - return metadataMap[metadataKey] = MetadataValue
##### MetadataKeys(P)
- 返回OrdinaryMetadataKeys(O,P)
    - 判断P
    - 使ownKeys为OrdinaryOwnMetadataKeys(O,P)
    - 使parent为o.GetPrototypeOf
    - 如果parent为null, 返回ownKey
    - 使parentKeys为O的OrdinaryMetadataKeys中属性为P的keys
    - 使ownKeysLen为ownKeys的长度(length)
    - 如果ownKeysLen为0, 返回parentKeys
    - 使parentKeyLen为parentKeys的长度
    - 如果parentKeysLen为0, 返回ownKeys
    - 使set为一个新的Set对象
    - 使keys为长度为0的数组
    - 让k = 0
    - 遍历ownKeys中的所有的key
        - 使hasKey为set中是否包含key
        - 如果hasKey为false, 
            - 使Pk为ToString(k)
            - set添加一个key
            - 使defineStatus为CreateDataProperty(keys, Pk, key) 即keys[Pk] = key
            - 断言defineStatus为真, 即设置成功
            - k加1
    - 遍历parentKeys中的key
        - 与ownKey使一个逻辑, 最后k依然加1
    - 设置keys的长度为k
    - 返回keys
#### OwnMetadataKeys(P)
- OrdinaryOwnMetadataKeys(O,P)
    - 断言P
    - 使keys为长度为0的数组
    - 使MetadataMap为GetOrCreateMetadataMap(O, P, false)
    - 如果metadataMap为undefined, 返回keys
    - 使keysObj为MetadataMap['keys']
    - 使iterator为keysObj的迭代器
    - 使k为0
    - 重复执行
        - 使Pk为k的字符串形式
        - next使为 iterator.next
        - 如果next为false
            - setStatus为 keys['length'] = k, 如果赋值失败, 则报错
            - 断言setStatus是否为真
            - 返回keys
        - 使nextValue为iteratorValue.next
        - 使defineStatus为keys[Pk] = nextValue
        - 如果defineStatus使一个被打断的状态, 该iterator close掉并且返回完成态defineStatus
        - k增加1
#### DeleteMetadata(MetadataKey, P)
- 断言P
- 使metadataMap为GetOrCreateMetadataMap(O,P,false)
- 如果metadataMap是undefined, 返回false
- 返回metadataMap['delete'] = MetadataKey
### 反射 - Reflection
这里其实大部分就是使用之前已经定义好的内部方法来把API暴露出来
#### Metadata Decorator Functions 元数据装饰器函数
一个元数据装饰器函数是一个匿名的内建函数, 以 MetadataKey以及MetadataValue为内部slots中
当一个元数据装饰器函数F(target, key)被调用时, 以下步骤会发生:
- 断言: F有MetadataKey内部slot, 且类型为基本类型, 或undefined
- 断言: F有MetadataValue内部slot, 且类型为基本类型, 或undefined
- 如果 Type(target) 不是一个对象, 抛出一个TypeError的错误
- 如果key时一个undefined并且 IsPropertyKey(key) -- 不是target的属性, 抛出一个TypeError的错误.
- 使metadataKey成为F MetadataKey内部slot值
- 使metadataValue成为F MetadataValue内部slot值
- 运行target.\[\[DefineMetadata\]\](metadataKey, metadataValue, target, key)
- 返回undefined 

#### Reflect.metadata(metadataKey, metadataValue)
当metadata函数被metadataKey和metadataValue作为参数调用时, 下列步骤会发生:
- 使decorator为新的内建函数, 就像定义的Metadata Decorator Functions
- 设置decorator的MetadataKey为metadataKey
- 设置decorator的MetadataValue为metadataValue
- 返回decorator

#### Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey)
- 如果target不是对象, 抛出错误
- 返回target的defineMetadata方法调用, 参数为(metadataKey, metadataValue, propertyKey)

#### Reflect.hasMetadata(metadataKey, target, propertyKey)
- 如果target为对象, 抛出错误
- 返回target的HasMetadata方法调用, 参数为(metadataKey, propertyKey)

#### Reflect.hasOwnMetadata(metadataKey, target, propertykey)
- 如果target为对象, 抛出错误
- 返回target的HasOwn方法调用, 参数为 (metadataKey, propertyKey)

#### Reflect.getMetadata(metadateKey, target, propertyKey)
- 如果target为对象, 抛出错误
- 返回 target的GetMetadata方法, 参数为(metadataKey, propertyKey)

#### Reflect.getOwnMetadata(metadataKey, target, propertyKey)
- 如果target为对象, 抛出错误
- 返回target的GetOwnMetadata(Metadata, propertyKey)

#### Reflect.getMetadataKeys(target, propertyKey)
- 如果target为对象, 抛出错误
- 返回target.GetMetadataKeys(propertyKey)

#### Reflect.getOwnMetadataKeys(target, propertyKey)
- 如果target为对象, 抛出错误
- target.GetOwnMetadataKeys(propertyKey)

#### Reflect.deleteMetadata(metadataKey, target, propertyKey)
- 如果target为对象, 抛出错误
- target.DeleteMetadata(metadataKey, propertyKey)





