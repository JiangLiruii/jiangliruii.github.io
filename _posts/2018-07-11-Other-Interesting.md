---
layout:       post
title:        "Other Interesting Things"
subtitle:     "积小流以成江河"
date:         2018-07-11 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   '/img/back/interesting.jpg'
catalog:      true
multilingual: false
tags:
    - Interesting
---

有时候看到一些有趣的东西,又不知道放在哪里,也担心过目即忘,索性开一个文,用于记录点滴,不定期整理成单独文章.

## [DOMContentLoaded vs Load Event](https://www.sitepoint.com/performance-auditing-a-firefox-developer-tools-deep-dive/)

`DOMContentLoaded`在HTML document 完成加载和实例化后(parsed)触发,不包括CSS stylesheet, images 和 frames.

`load`是当HTML Document andAll Associated stylesheets, images and frames are completely loaded.是在DOMContentLoaded之后,包括了其不包括的内容.

## Network Timings
![](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2017/12/1512623728network-timings-1024x188.png)

### Blockd: 网络连接的排队时间
### DNS resolution: 解析服务器的主机名的时间
### Connecting: 打开TCP连接的时间
### TLS setup: TLS(Transport Layer Security)设置的时间(也有可能是SSL)
### Sending: 向服务器发送请求的时间
### Receiving: 从服务器获取请求的时间(如果有缓存的话为读取缓存的时间)
### Waiting: 客户端接到第一个字节数据前所花费的时间,在Chrome DevTool中是**TTFB(Time To First Byte)**

## 写一点关于react 生命周期的东西

|Mounting|Updating Component props|Updating Component state|Update Using|Unmounting|
| ----- | ----- | ----- | ----- | ----- |
|constructor||||
|componentWillMount||||
||componetWillRecieveProps|||
||shouldComponentUpdate|shouldComponentUpdate||
||componentWIllUpdate|componentWIllUpdate|componentWIllUpdate|
|render|render|render|render|
||componentDidUpdate|componentDidUpdate|componentDidUpdate|
|componentDidMount||||
|||||componentWillUnmount|

### 强制刷新 ‵this.forceUpdate()`

如果在render中使用的data不是state中的data,并且需要根据这个data的变化而变化,可以在data改变时调用该方法.但是应该尽力避免使用,因为会造成组件的不纯净.

### `componentWillMount()`

通常不会将网络请求(ajax,fetch)等异步操作放进这个生命周期中,因为如果异步获取数据的耗时小于render的耗时,将会导致还未mount的元素进行rerender,其二是在非浏览器环境中(比如server)也会触发,有副作用.

### 跟update相关方法:(具体流程可参考上述表格) 
- componentWillRecieveProps
- shouldComponentUpdata
- componentWillUpdate
- componentDidUpdate

```js
// because react will not rerender when props change, so you can use this method to observe props
componentWillRecieveProps(newProps) {
    this.setState({
        opacity: (newProps.visible) ? 1 : 0,
    })
}
// control whether to trigger rerender by checking truely change states or not
shouldComponentUpdate(newProps, newState) {
    return this.state.opacity !== + newProps.visible;
}
```

### Unmount事件

```js
window.addEvenListener('beforeunload', (e) => {
    let confirmMessage = '确定退出吗?';
    e.returnValue = confirmMessage; // Chrome 34+
    return confirmMessage; // Chrome 34-
});
```


## React Events
### Bind
```js
class SaveButton extends React.Component {
    constructor() {
        super();
        this.handleSave = this.handleSave.bind(this);
    }
    handleSave() {

    }
    render() {
        return <button onClick={this.handleSave} >Save</button>
    }
}
``` 

### Nullifying a Syntax Event不能在异步中获取事件
```js
class Mouse extends React.Component {
    handleMouseOver(e) {
        console.dir(e.target);
        setTimeout(()=> {
            console.log(e.target); // cannot access to e asyncly, you need use e.persist()
        })
    }
}
```

### 如何理解stateless
```js
// Statefulness
class ClickButton extends React.Component {
    constructor() {
        super();
        this.state = {
            counter: 0
        }
    }
    clickHandler() {
        this.setState({
            counter: this.state.counter + 1
        })
    }
    render() {
        return <button onClick={this.clickHandler.bind(this)} >{this.state.counter}</button>
    }
}

```

```js
// stateless
class ClickButton extends React.Component {
    render() {
        return <button onClick={this.props.clickHandler} >{this.props.counter}</button>
    }
}

// outside Component, this is statefulness, makes pure in more level
class Content extends React.Component {
    constructor() {
        super();
        this.clickHandler = this.clickHandler.bind(this);
        this.state = {
            counter: 0
        }
    }
    clickHandler() {
        this.setState({
            counter: ++this.state.counter
        })
    }
    render() {
        <ClickButton clickHandler={this.clickHandler} counter={this.state.counter} />
    }
}

```


## Form in React: 受控与非受控

React推荐使用受控组件,这样可以直接通过this.state来控制组件的状态.
主要通过两种方式实现控制:

1 value
2 onChange(推荐使用onChange,因为onChange与HTML中的onChange不一样,更类似与HTML中的onInput,即每次修改都会触发),可在此时进行value的校验等
```js
<input value={this.state.value} onChange={this.onChangeHandle} />
```
input的可控属性
- text
- password
- radio
- checkbox
- button

有两个点想要强调,

1 因为checkbox可以多选,所以在setState的时候要注意使用Object.assign(this.state)或JSON.parse(JSON.stringify(this.state))或(Array.from(this.state)).slice()

2 需要表达false的时候不能使用`checked=false` 而是需要使用 `checked={false}`因为会将其视为'false'子符串,然后转型为true.

其他form标签:

- select option
- textarea

需要注意的点:

- multiple select的实现
```js
<select multiple={true} value={['metor', 'react']}>
    <option value='metor'>Metor</option>
    <option value='react'>React</option>
    <option value='node'>Node</option>
</select>
```

### Uncontrol组件的用法

1 纯Uncontrol
```js
<input type='text' />
```

2 处理onChange,可以将change的值放入state中,用于submit或其他用途.但不更改其value属性.
```js
<input type='text' onChange={this.onChangeHandler}>
```

那么遇到第一种情况,纯Uncontrol,如何在submit的时候拿到value呢?
```js
class UncontrolComponent extends React.Component {
    constructor() {
        super()
        this.onSubmitHandle = this.onSubmitHandle.bind(this)
    }
    onSubmitHandle() {
        const value = React.findDOM(this.refs.input).value
        fetch(url, {method:'POST',body:{value}})
    }
    render() {
        return <form>
            <input ref='input'/>
            <button onClick={this.onSubmitHandle}>        
        </form>
    }
}
```

## HOC high-order-component
HOC可以将component作为参数传入,最后返回一个封装好的component, 其本质即为factory function.

优点也就是可以保证component的分离,presentational component 和 container component, presentational只负责pure component, 不包含state, 而container component借助HOC拿到它需要的component,并组合起来.

```js
// in container component
const factoryComponent = Component => {
    class _factoryComponent extends React.Component {
        constructor() {
            super();
            this.state = {
                label: 'Run'
            };
            this.onClickHandler = this.onClickHandler.bind(this);
        }

        onClickHandler() {
            let iframe = document.getElementById('frame').src = 'xxx.com'
        }

        render() {
            return <Component {...this.state} {...this.props} />
        }
    }
    _factoryComponent.displayName = 'ExtendComponent';
    return _factoryComponent;
}

render() {
    let ExtendedButton = factoryComponent(Button);
    let ExtendedLink = factoryComponent(Link);
    let ExtendedLogo = factoryComponent(Logo);
    return <div>
    <ExtendedButton />
    <ExtendedLink />
    <ExtendedLogo />
    </div>
}

// in presentational components
class Button extends React.Component {
    render() {
        return <button onClick={this.props.onClickHandler} >{this.props.label}</button>
    }
}
// transform to function
const Button = props => <button onClick={props.onClickHandler} >{props.label}</button>

const Logo = props => <img src='logo.png' href='#' onClick={props.onClickHandler} />

const Link = props => <a onClick={this.onClickHandler} href='#'>{props.label}</a>
```

## 滚动条宽度

macOS: 15px
Windows: 17px --> Chrome, Firefox, IE  16px --> Edge  15px --> Opera

### 如何测量呢?

```js
// 定义外盒子
const outer = document.createElement('div');
// 定义内盒子
const inner = document.createElement('div');
// 定义overflow
outer.style.overflow = 'scroll';
// 添加外盒子到body中
document.body.appendChild(outer);
// 添加内盒子到外盒子中
outer.appendChild(inner);
// 根据宽度差计算滚动条宽度
const scrollbarWidth = outer.offsetWidth - inner.offsetWidth;
// 移除外盒子
document.body.removeChild(outer);
```
注意以上代码只能检测原始的滚动条宽度,无法检测在css中设置过的滚动条宽度, 而且有DOM操作, 对性能也有所影响.

### 更好的解决办法是隐藏滚动条:

```css
/* for chrome safari opera */
.container::-webkit-scrollbar {
    display: none;
}
/* for ie, edge */
.container {
    -ms-overflow-style: none;
}
```
但是对于Firefox是没有办法隐藏滚动条的.

### 其他对滚动条的css设置:
```css
body {
    scrollbar-face-color: blue;
}
/* webkit前缀支持 */
::-webkit-scrollbar {
    width: 8px;
}
::-webkit-scrollbar-thumb {
    background-color: #fff;
    border-radius: 4px;
}
```

### 流畅的操作体验
对于滚动而言,最常见的就是登录页导航, 通常是通过锚点来完成:
```html
<a href="#section">Section</a>
```
只是用一行js便可实现平滑滚动
```js
elem.scrollIntoView({
    behavior: 'smooth',
})
```
还有一个css属性可以改变整个页面的滚动行为
```css
html {
    scroll-behavior: smooth;
}
```

### 粘性css(`position: sticky`)

![](https://mmbiz.qpic.cn/mmbiz_gif/btsCOHx9LAOlAlnq7uds6icTwS4Mhsoyl5n0jvL6vxlIE0eiaH2uKzwToExWiavbtXdGTHfzpWouGS5ickdCF0RObQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

```css
.element {
    position: sticky;
    bottom: 50px;
}
```
上述代码表示在viewport中,当element距viewport的bottom小于50px时会将position变为fixed, 实现距离底部始终为50px, 直到当距离bottom大于50px时恢复position为relative的状态.`left, right, top`同理.

### 滚动的函数节流

通常我们使用`addEventListener`去处理
```js
window.addEventListener('scroll', () => {
    const scrollTop = window.scrollY;
    // dosomething with scrollTop
});
// 使用函数节流
window.addEventListener('scroll', throttle(() => {
    const scrollTop = window.scrollY;
    // dosomething with scrollTop
}))
// 定义节流函数,默认1秒触发一次
function throttle(action, wait = 1000) {
    let time = Date.now();
    return function () {
        if ((time + wait - Date.now()) < 0) {
            action();
            time = Date.now();
        }
    }
}
// 或者使用requestAnimationFrame
function throttle(action) {
    let isRunning = false;
    return function() {
        if (isRunning) return;
        isRunning = true;
        window.requestAnimationFrame(()=> {
            action();
            isRunning = false;
        })
    }
}
```
### 在视窗中显示

当图片进行懒加载活无限滚动时, 需要确定元素是否出现在视窗中.可以使用`element.getBoundingClientRect()`

```js
window.addEventListener('scroll', () => {
    const rect = elem.getBoundingClientRect();
    const inViewport = rect.bottom > 0 && rect.right > 0 && rect.left < window.innerWidth && rect.top < window.innerHeight
})
```
但是会触发回流(reflow), 具体情况请参考 [layout thrashing](https://lorryjiang.info/2018/07/01/Layout-Thrashing/#%E6%80%BB%E7%BB%93)
影响性能, 虽然也可以通过throttle函数节流,但是也只能减小一些影响,有没有更好的办法呢?

使用 [`IntersectonObserver()`](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)这一API来解决该问题,允许追踪目标元素与祖先元素或视窗的交叉状态.只要有部分元素出现在视窗中,哪怕只有一像素都会触发回调函数.
```js
const observer = new IntersectionObserver(callback, option);
observer.observe(element)
```
### 滚动边界

如果某个下拉列表是可以滚动的,需要注意一些连锁滚动相关的问题.比如:
当用户滚动到下拉列表的末尾时,后续再滚动,整个页面都会滚动.
![](http://p799phkik.bkt.clouddn.com/roll.gif)
当滚动到底部时可以改变页面的`overflow`属性活在滚动元素的滚动处理函数中取消默认行为.记住处理的是 `wheel` 而不是 `scroll` 事件

```js
function handleOverscroll(event) {
    const delta = -event.deltaY;
    if (delta < 0 && elem.offsetHeight - delta > elem.scrollHeight - elem.scrollTop) {
        elem.scrollTop = elem.scrollHeight;
        event.preventDefault();
        return false;
    }
    if (delta > elem.scrollTop) {
        elem.scrollTop = 0;
        event.preventDefault();
        return(false);
    return true;
    }
}
```
不幸的是,移动端的表现并不好,因为有下拉刷新手势,会阻碍上述代码的执行.

css可以通过 `overscroll-behavior`解决问题
```css
.element {
    overscroll-behavior: contain;
}
```
 ### 惯性滚动

 ```css
 .element {
     -webkit-overflow-scrolling: touch;
 }
 ```
 只用于支持webkit的浏览器,只适用于触屏设备

 而且如何处理`touchstart` 和 `touchmove` 事件存在的性能问题, 为了让滚动变得平滑,需要执行Event.preventDefalut()以取消默认行为,有时仍可能需要花费时间来等待事件处理函数执行完毕.

有一个passive event listeners可以用
```js
element.addEventListener('touchstart', e => {}, {passive: true});
```
以下是使用了passive event listener和未使用的对比视频,[CNN网页对比_from_youtube](https://www.youtube.com/watch?v=NPM6172J22g)

## 感谢
> [Evil Martians’ team](https://evilmartians.com/chronicles/scroll-to-the-future-modern-javascript-css-scrolling-implementations)




## 相对路径
### ../src ./src /src src 的区别

- ../src是父级目录中的src
- ./src是同级目录中的src
- /src是根目录下的src(与package.json同级)
- src与./src一样,表示同级目录

### js与css中的相对定位

```css
@font-face {
    font-family: 'my special font';
    src: url('../a/b/c');
}

a {
    font-family: 'my special font';
}
```
这里url中的相对定位是相对于**css文件**所在目录.

```js
// a.js
$.get('../a.html', () => console.log('got a'));
```
通过script标签在html引用
```html
<script src='a.js'></script>
```
在a.js中的相对定位是**基于html**的位置

当然现在大多数js是通过模块引用,而不是script标签, 所以基本上js也是基于自身定位,但请注意在webpack中引用打包后的资源的位置,而不是打包前开发环境中的位置.比如:
```js
// webpack
  copy: [
    {
      from: './public/audio',
      to: './dist/audio',
    }
  ],
```
打包后的index.js与dist同级,所以要获取audio直接使用./audio即可. 注意需要使用`copy-webpack-plugin`插件

### webpack打包后路径引用

```js
// webpack.config.js 看看这两者的差别
module.exports = {
    publicPath: './',
    publicPath: '/',
}
```
如果以http协议访问是没有区别的,两者都一样, 但是如果是file协议进行访问, 那么前者可正常, 后者会被引用到根目录, 导致无法访问的问题, 所以建议配置'./'

8.5 今天在修bug的时候合作方提了一个需求: 图片太长,低端机的拖动较为卡顿.

在已经使用tinypng进行压缩不能再对其大小进行限制的时候想到了有没有对滑动属性的支持,后来google之后果然发现一个属性`--webkit-overflow-scrolling: touch` 使用这个属性就可以让图片在手指滑动完毕之后依然进行惯性的滑动,以此来确保滑动的流畅性.

在全局中`<body>`或长高均为100%的元素中是默认会应用该属性,但是局部滑动却不会应用.所以在需要局部刷新时可对此属性进行优化.

除此之外还有其余的属性能够优化滑动效果:

```css
.img {
-webkit-overflow-scrolling: touch;
-webkit-transform: translateZ(0px);
-webkit-transform: translate3d(0,0,0);
-webkit-perspective: 1000;
}
```


## 参数的解构

一直用的是ES6的解构语法,比如:
```js
let a = function(..args) {
    console.log(...args)
}
a(1,2,3,4) // 1 2 3 4
```

但是如果是ES5的话应该如何使用呢?最近看书发现了一个用高阶函数进行封装达到解构效果的方法.

```js
let a = fn => Function.apply.bind(fn, null);
a((b,c) => console.log(b,c))([1,2]) // 1,2

// 实例:
function getY(x) {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve(x * 3), 1000);
    })
}
function foo(bar, baz) {
    const x = bar * baz;
    // 返回两个promise
    return [Promise.resolve(x), getY(x)]
}
Promise.all(foo(10,20)).then(a((b, c) => console.log(b,c))) // 200, 600

```
等价于
```js
function a([b,c]) {
    console.log(b,c)
}
a([1,2,3]) // 1,2
// 上例可改写为:
Promise.all(foo(10,20)).then(([b,c]) => console.log(b,c)) // 200, 600
```

### 用户输入验证

监听input事件

```html
<input id="test"/>
```
```js
// 匹配emoji表情和颜文字
function validate_string() {
  return /(\ud83c[\udf00-\udfff])|(\ud83d[\udc00-\ude4f])|(\ud83d[\ude80-\udeff])/g;
}
// 只允许汉字,字母,数字输入
function validate_variable_name() {
return !/^[\u4E00-\u9FA5A-Za-z][\u4E00-\u9FA5A-Za-z0-9]*$/g;
};

document.getElementById('test').addEventListener('input', (e) => {
    e.target.value.replace(validate_string(), '');
    e.target.value.replace(validate_variable_name(), '');
    return false;
})
```
在往深一步, 国人在输入拼音的时候有一个中间态, 即拼的过程, 姑且成为间接输入也会触发input事件.

还有两个事件, `compositionstart` 间接输入时触发 和 `compositionend`间接输入结束时,即用户选词时触发, 那么就可以通过一个inputlock进行状态锁定

```js
let inputLock;
document.getElementById('test').addEventListener('compositionstart', () => {
    inputLock = true;
})
document.getElementById('test').addEventListener('compositionend', () => {
    inputLock = false;
    dosomething(e.target);
})
document.getElementById('test').addEventListener('input', (e) => {
    if (!inputLock) {
        dosomething(e.target);
        return false;
    }
})
```

### 关于可视区域

判断元素是否在可视区域
`domElemen.offsetTop < window.innerHeight + documen.body.scrollTop`
domElement.offsetTop: DOM元素距离顶部的值
window.innerHeight: 视窗高度, 包括内容, 边框, 滚动条
window.scrollY: window滚动的高度

窗口高度: 
window.outerHeight: 整个浏览器窗口的大小, 包含窗口标题, 工具栏, 状态栏
documentElement.clientHeight: 视窗高度, 不包括整个文档的滚动条,也不包括元素的边框和滚动条
document.body.clientHeight: 整个body的高度, 包含滚动条.

滚动高度:
clientHeight: 内部可视区域大小
offsetHeight: 可视区域大小, 包含border和scrollbar = document.body.clientHeight
scrollHeight: 元素内容高度, 包括overflow
scrollTop: 元素内容向上滚动了多少像素

举点例子吧:
```js
// 获取整个body高度
document.documentElement.offsetHeight = document.body.clientHeight = document.body.offsetHeight = document.body.scrollHeight = document.documentElement.scrollHeight
// 获取当前视窗scroll值
window.scrollY = document.documentElement.scrollTop
// 获取当前窗口的高度
document.documentElement.clientHeight = window.innerHeight
```
### 要知道在html中每个元素都是被绑定在一个矩形中的,也就是盒型模型, 如何获取这个大小和位置呢?
`Element.getBoundingClientRect()`

包含了以下只读属性:
- width
- height
以下单位为像素, 且相对viewport的左上角
- left
- top
- right
- bottom

### 判断某个元素是否进入视图 IntersectionObserver

传统的做法是监听scroll事件, 调用getBoundingClientRect(), 得到相对于左上角的坐标, 再判断是否在窗口内. scroll很消耗性能

这个API检测目标元素是否与当前视口产生了交叉区, 所以也叫做交叉观察器.

```js
/*
* callback, 会被调用两次, 一次进入视图, 一次离开视图 
* entries是一个数组, 监听几个dom就有几个成员, 每一个成员都有以下属性: 1 time 发生变化的时间 2 target 目标元素 3 rootBounds 根元素矩形信息 4 boundingClientRect, 目标元素区域信息 5 intersectionRect 目标元素与视图窗口(或根元素)的交叉区域信息 6 intersectionRatio 目标元素的可见比例 即intersectionRect / boundingClientRect, 完全可见时为1, 不可见为0
* options
*/
let io = new IntersectionObserver(entries => {
    entries.forEach(entry => {
        console.log(entry)
    })
})

// 开始观察, 指定DOM节点
io.observe(dom)
// 停止观察
io.unobserve(dom)
// 关闭observer
io.disconnect()
```
关于intersectionRatio, 阮一峰有一幅图: ![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016110202.png)

这个拿来有什么用呢??

### 惰性加载 (lazy load), 只有用户向下滚动, 进入视图才加载

```js
let observer = new IntersectionObserver(entries => {
    entries.forEach(entry => {
        let container = entry.target;
        let content = container.querySelector('template').content;
        container.appendChild(content);
        observer.unobserve(container);
    })
})

Array.from(document.querySelectorAll('.lazy-loader')).forEach(item => observer.observe(item))
```
### 无限滚动
```js
let intersectionObserver = new IntersectionObserver(entries => {
    if(entries[0].intersectionRatio <= 0) return;
    loadItems(10);
    console.log('load new items');
})
intersectionObserver.observe(
    document.queryselector('.scrollerFooter')
)
```
当页面滑动到尾页栏时, 加载新的条目在尾页栏前, 这样就可以让observer重复使用.

### 在参数中除了callback之外还有option对象:
#### threshold属性
```js
new intersectionObserver(entries => {}, {
    threshold: [0,0.25,0.5,0.75,1]
})
```
效果为:![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016110202.gif)
#### root属性: rootMargin
目标元素除了随着默认的窗口滚动, 还可能在container中滚动, 比如iframe.容器元素必须是目标元素的祖先节点.
```js
let opt = {
    root: document.querySelector('.container'),
    // 用来扩展或缩小rootBounds这个矩形的大小
    rootMargin: '500px, 0px',
}
```
### intersectionObserver优先级很低,只能在浏览器空闲后执行.


## 关于terminal操作指令:

### 遇到端口占用时:
`lsof -i tcp:[port]` 显示占用端口pid
`kill -9 [pid]` 退出对应pid, 释放端口


## javascript点滴

### new.target

```js
// 函数强制返回new调用实例:
function foo(name) {
    if (!new.target) {
        return new foo(name);
    }
	var self = this;
	self.name = name;
}
// 抛出一个错误:
function foo(name) {
    if (!new.target) {
        throw Error('must called with new');
    }
	var self = this;
	self.name = name;
}
// new.target.name 属性, 通常情况下就是函数名
function foo() {
    console.log(new.target.name)
}
const a = new foo();// foo
// 如果是类的情况
class A {
    constructor() {
        console.log(new.target.name)
    }
}
class B extends A {
    constructor() {
        super();
    }
}
const a = new A();// A
const b = new B();// B
// 可以看到这个target就是new时候的对象.
```