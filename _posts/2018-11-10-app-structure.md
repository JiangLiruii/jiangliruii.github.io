---
layout:       post
title:        "App structure"
subtitle:     "自制轻型应用框架"
date:         2018-11-10 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/structure.jpg'
catalog:      true
multilingual: false
tags:
    - web
---
# 创建大型应用时, 必不可少的需要一个程序架构

## 本文主要包括以下部分
- 单页应用架构
- MV*
- 数据模型(model)和数据模型集合(collection)
- 单数据视图(item view) 和 集合视图 (collection view)
- 控制器(controller)
- 事件(event)
- 路由(router) 和 hash导航
- 中介器(mediator)
- 客户端渲染和Virtual DOM
- 数据绑定和数据流
- Web组件和shadow DOM
- 选择一个MV*框架

## 单页应用(SPA)框架

### 一个SPA就是一个Web应用, 他所需的资源(HTML, CSS, JS)在一次请求中就加载完成, 或者更准确的说是在数据初始化之后, 就不会再有刷新(重新加载).具体实现方式:

- Ajax, 通过请求之后的success回调, 以xml或json为响应返回数据, 对DOM进行操作

```js
function handleClick(product_id) {
  $.ajax({
    method: 'GET',
    url: `api/product_detail/${product_id}`,
    dataType: 'json',
    success: (data_json) => $('#product_container').innerHTML = data_json.html,
    error: (e) => throw Error(e)
  })
}
```
#### 像现在流行的框架 react, vue, 都是单页, 他们的本质都是于此.

## MV*架构
### SPA中很多在传统服务端的工作迁移到前端, 增加了js代码的数量, 如何更好的组织代码?于是有了不同的设计模式.需要重点关注的是MVC(Model-View-Controller) 和其一些衍生版本MVVM(Model-View-ViewModel), MVP(Model-View-Presenter)

## MV*框架中的组件和功能

### model 
储存数据的组件, 通常从HTTP API请求过来并显示在view上.比如使用一个较为流行的Backbone.js, model类需继承自Backbone.Model类
```ts
class TaskModel extends Backbone.Model {
  public created:number;
  public completed:boolean;
  public title:string;
  constructor () {
    super()
  }
}
```
该model继承了一些方法, 可以于网络服务进行通信, 使用fetch方法请求数据, 并将其设置到model中.

### collection
用于展示一组model.
```ts
class TaskCollection extend Backbones.Collection<TaskModel> {
  public model:TaskModel;
  constructor() {
    this.model = TodoModel;
    super();
  }
}
```

### item view
负责在model中的数据渲染成HTML.通常依赖构造函数, 属性, 设置中传入model, 模板或容器
- model和模板用来生成HTML
- 容器通常是一个DOM selector, 将HTML插入
```ts
class NavBarItemView extends Marionette.ItemView {
  constructor(options:any = {}) {
    options.template = '#navBarItemViewTemplate';
    super(options)
  }
}
```

### collection view
它于view类似于collection于model, 是一个集合的概念. collection view 迭代collection里面存储的model, 使用item view渲染它, 然后追加到容器尾部.
但是这样容易造成性能瓶颈, 更好的实现方式是使用一个item view和属性为数组的model, 然后使用{{ # each }} 语句在view的模板中将这个列表渲染出来, 而不是为collection的每一个元素都渲染一个view

```ts
class SampleCollectionView extends Marionette.CollectionView<SampleModel> {
  constructor(options:ang = {}) {
    super(options)
  }
}
var view = new SampleCollectionView({
  collection,
  el: $('#divoutput'),
  childView:SampleView
})
```

### Controller
负责管理特定的model相关的view声明周期, 职责是实例化model和collection, 并于相关的view联系起来, 在将控制权交给其他controller前销毁他们.
MVC的应用交互是通过组织controller和它的方法, 这些方法和用户的行为一一对应.
```js
class LikeController extends Chaplin.Controller {
  public beforeAction() {
    this.redirectUnlessLoggedIn();
  }
  public index(params) {
    this.collection = new Likes();
    this.view = new LikesView({collection: this.collection})
  }
  public show(params) {
    this.model = new Like({id: params.id});
    this.view = new FullLikeView({model: this.model})
  }
}
```
LikeContrller有两个方法, index和show, 在这两个方法执行前, beforeAction都会执行.

## 事件

指程序发现的行为或发生的事情, 可能会被程序处理. MV*通常由两种事件:
- 用户事件: 程序允许用户通过触发和处理事件的形式沟通, 如单击, 滚动屏幕, 提交表单. 用户事件常在view中处理
- 程序事件: 应用自身也可以触发和处理一些事情, 比如view后的onrender事件, JavaScript的 document.onload事件等等

程序事件是遵循SOLID原则中(单一职责, 开放封闭, 里氏替换, 依赖倒置, 接口分离)的开/闭原则的一个好的方式, 可以使用事件来允许开发者扩展框架, 而不需要对框架左任何修改

## 路由和hash导航
路由负责观察 url的变化 并将程序执行切换到对应的controller的方法上. 主流框架都是用了hash导航混合技术.使用的就是H5的history API, 在不重载的情况下变更url.

在SPA中, 链接通常包含了一个hash(#)字符, 这个原来是被设计为定位到某个DOM元素上, 但选择被用作**无刷新导航**.

```ts
class Route {
  public controllerName:string;
  public actionName:string;
  public args:Object[];
  
  constructor(controllerName:string, actionName, args:Object[]) {
    this.controllerName = controllerName;
    this.actionName = actionName;
    this.args = args;
  }
}

// 一个最基本的路由
class Router {
  private _defaultController:string
  private _defaultAction:string
  constructor(defaultController:string, defaultAction:string) {
    // 设置默认值, 其实也可以使用ES6的特性传默认参数
    this._defaltController = defaultController || 'home';
    this._defaultAction = defaultAction || 'index';
  }
  public initialize() {
    // 观察用户url改变
    $(window).on('hashchange', () => {
      var r = this.getRoute();
      this.onRouteChange(r)
    })
  }
  
  // 读取url
  private getRoute() {
    return this.parseRoute(window.location.hash)
  }
  // 解析url
  private parseRoute(hash:string) {
    var comp, controller, action, args, i;
    if (hash[hash.length - 1] === '/') {
      hash = hash.substring(0, hash.length - 1)
    }
    comp = hash.replace('#', '').split('/')
    controller = comp[0] || this._defaultController
    action = comp[1] || this._dedfaultAction
    args = []
    for (i = 2; i < comp.length; i++) {
      args.push(comp[i])
    }
    return new Route(controller, action, args)
  }

  private onRouteChange(route:Route) {
    // 在此处执行控制器
  }
}
```
URL的命名规则遵循: 
`#controllerName/actionName/arg1/arg2/arg3`
比如 `<a href="#home/index">`当点击的时候, `r = new Route('home', 'index', [])`

## 中介器

通常实现于发布/订阅设计模式(pub/sub), 可以让模块之间不用相互依赖, 通过事件通信, 而不是直接使用程序中的其他部分
中介器可以让开发者轻松的进行扩展, 而不需要对框架进行任何改动, 可以很好的遵循SOLID原则.
```ts
interface Imediator {
  publish(e:IAppEvent):void;
  subscribe(e:IAppEvent):void;
  unsubscribe(e:IAppEvent):void;
}
// 接上一个示例
class Router {
  private onRouteChange(route:Route) {
    this.mediator.publish(new AppEvent('app.dispatch', route, null))
  }
}
```
上面的代码避免了直接调用controller, 而是使用中介器发布一个事件
Router --> Mediator --> Controller

## 调度器
`app:dispatch`是否引起了你的重视?app.dispatch指向一个调度器的东西, 意味着路由在向调度器发送事件而不是controller

```ts
class Dispatcher {
  public initialize() {
    this.mediator.subscribe(
      new AppEvent('app.dispatch', null, (e:any, data?any) => this.dispatch(data));
    )
  }
  // 创建和销毁controller实例
  private dispatch(route:IRoute) {
    // 销毁旧的controller
    // 创建新的controller
    // 通过中介器触发控制器的action
  }
}
```
1 Router --> 2 Mediator <--> 3 Dispatcher
                 |
                 |
            4 Controller

## 客户端渲染和Virtual DOM, 分两个方面

### 何时渲染
客户端渲染需要一个模板和一些数据取生成HTML, 但还需要注意一些性能方面的细节. 操作DOM是SPA性能瓶颈主要原因之一.
- 使用定时器检测变更 --- 脏检测
- observable model

observable的实现比使用定时器更高效, 因为observable仅在变更发生的时候触发, 而定时器则是无条件按时间间隔执行.

### 如何渲染
- 直接操作DOM
- 在内存中操作被成为Virtual DOM的DOM映射.
显而易见, Virtual DOM更加的高效, 因为js的内存操作更迅速

## 用户界面数据绑定
UI数据绑定可以简化图形化界面开发的设计模式, 将一个UI元素和程序的model绑定在一起, 一个绑定会将两个属性关联在一起, 当其中一个改变时, 另外一个的值也将自动更新. 绑定可以将同一对象或不同对象上的元素联系在一起.

### 绑定方式
- 单向数据绑定--仅能单向传播变更
Model
      \
       \
        \
         -->one-time merge --> View
        /
       /
      / 
Template

- 双向数据绑定
     Template
       |
       |  compile
      \|/
 -----View-----
/|\           |
 |            |
 |           \|/
 -----Model----
### 数据流
Flux提出了一个新概念: 单向数据流--> 一次变量值的变更, 都会导致依赖该变量的其他变量重新计算自己的值, 然后更新render在View中. 具体的实现方式为:
- 所有的Action直接发送到Dispatcher中, 然后将执行流交给Store
- Store用来储存和操作数据, 类似于MVC中的model, 每当数据被修改时, 会传递给view
- View负责将数据渲染成HTML并处理事件(Action). 如果一个事件需要修改一些数据, View会将这个Action送入到Dispatch中, 而不是直接对model进行修改. 这是与双向数据绑定的区分点.

单向数据流可以让程序的执行流非常清晰且可预测.

### Web component 和 shadow DOM
可以重用的UI组件, 允许用户自定义HTML, 比如自定义一个新的标签<map>来显示地图. Web Component 可以单独引入自己的依赖, 并且使用一种叫shadow DOM的客户端模板渲染HTML
shadow DOM允许在Web component 中使用HTML, CSS, JS, 可以避免模块之间的HTML, CSS, JS的冲突

Polymer是实现了真正的Web component, 但是React使用的是可复用的UI组件并不是真正的Web component, 因为没有使用到Web component相关技术(如上述)

## 从零开始实现一个MVC框架
材料准备:
- controller: 初始化view 和 model, 完成初始化后将执行流交给一个或多个model
- view: 加载和编译模板, 一旦模板编译完成, 等待model传入数据, 编译传入的数据, 并插入到DOM中, 同时也负责绑定和解绑ui事件
- model: 负责与HTTP API通信, 并在内存中维护数据. 设计数据的格式化和数据的增减. 一旦完成对数据的操作, 将传递到一个或多个view中
- router: 路由观察浏览器URL的变化,并在变更时创建一个route实例, 通过程序事件传递给dispatcher
- route: 被用来表示一个URL, URL的命名规则可以指明那个controller的方法在特定路由下被调用
- component: 程序的根组件, 负责初始化框架内所有的内部组件(包括中介器, 路由和调度器)
- event: 将信息从一个组件发送到另一个, 也可以订阅或取消订阅
- dispatcher: 接受一个Route实例, 用来指定依赖的controller, 如果需要会销毁上一个controller并新建, 一旦controller被创建, 执行流便会交给controller
- mediator: 负责程序中所有其他模块间的通信

路由 --> 程序组件 -------------
                    |       |
                    |       |
                    |       |
                   \|/     \|/ 
                   调度器  中介器
                    |
                    |
      Controller<----                                
      /       \  
     /         \
    /           \
  |/_           _\|
model   <----->  view <----模板

可以参见我的[github](https://github.com/JiangLiruii/SimpleStructure), 上面的commit详细记录了我进行编码的过程, 对每一个上述的材料都进行了编写.
