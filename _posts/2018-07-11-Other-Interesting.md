---
layout:       post
title:        "Other Interesting Things"
subtitle:     "积小流以成江河"
date:         2018-07-11 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/promise.jpg'
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