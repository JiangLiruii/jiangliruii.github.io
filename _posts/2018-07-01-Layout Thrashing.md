---
layout:       post
title:        "Layout Thrashing"
subtitle:     "你的一点点改变,却要改变整个的我"
date:         2018-06-21 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img:   'http://p799phkik.bkt.clouddn.com/promise.jpg'
catalog:      true
multilingual: false
tags:
    - JavaScript
---
# 起因
最近项目需要跟硬件打交道,硬件是用C++写的,需要实时接收硬件发送的数据,如果没有对硬件设置延时发送数据的话会导致前端页面无法及时响应,触发layout的回流(reflow),最终程序会卡死.

涉及到了性能优化.寻找程序中比较消耗性能的操作,最后发现了有一个队scroll的操作.目的是实现数据更新的滚屏.
```js
message_container.scrollTop = message_container.scrollHeight - message_container.clientHeight;
message_container.scrollLeft = message_container.scrollWidth - message_container.clientWidth;
```

# 结缘
刚好最近在看[Secrets of the JavaScript Ninja](https://www.manning.com/books/secrets-of-the-javascript-ninja-second-edition)的书中(强烈建议!)在讲DOM操作的时候提到需要最小化thrashing,隐隐约约觉得会是一个突破口

## 什么是Thrashing

在浏览器执行DOM写操作和读操作实际上浏览器是会对其进行一些布局性能的优化(perform layout optimizations),最简单的一个滚屏的例子,假设一个 message_container 作为装载信息的容器

```js
message_container.scrollTop = message_container.scrollHeight - message_container.clientHeight; // 测量时需更新layout
message_container.scrollLeft = message_container.scrollWidth - message_container.clientWidth; // 又一次测量,因为被设置过,所以需要再一次更新layout
```
看似上述代码十分简洁,两行搞定,但实际上不如以下操作
```js
const height = message_container.clientHeight;
const width = message_container.clientWidth;
const scroll_height = message_container.scrollHeight;
const scroll_width = message_container.scrollWidth; // 全部为测量,没有设置,所以只会更新一次layout的更新
message_container.scrollTop = scroll_height - height;
message_container.scrollLeft = scroll_width - width;
```
虽然只少了两次(scrollWidth, clientWidth)如果在多次循环的时候,差距会累积得越来越大.

## 现状

我的工作中就遇到了这个问题,因为需要跟硬件结合,硬件语言是c++,会通过端口触发`data`事件来发送数据,硬件发送的频率非常的快,最短可在1毫秒内发送数据,通过事件监听会很快的卡掉,会看到控制台有强制回流冲突的现象,因为进行layout的更新跟不上数据改变的速度,这是一个性能瓶颈.

## 改进

1 设置延时,先将数据保存,然后再在一定延时之后更新到layout中,延时时间设置为16ms(60fps).

2 使用thrashing优化,减少layout更新, 包括滚屏逻辑读写分离,innerText的定量清空.

3 在信息达到一定数量后进行清空

## 代码

```js
window.onload = () => {
  // 初始化串口
  serialport.SerialPort.initSocket('https://localhost:23321/sp')
  // electron相关
  const electron = require('electron');
  const { remote, ipcRenderer} = electron;
  const dialog = remote.dialog;
  const current_window = remote.getCurrentWindow();
  // 页面相关
  const message_container = $.select('.message_container')[0];
  const send = $.select('#send_value');
  const select_rate = $.select('#rate_select');
  const height = message_container.clientHeight;
  const width = message_container.clientWidth;
  // 设置全局变量
  let port, sp, need_scroll = true, arr = [], print_num = 0, timer;

  message_container.innerText = '';
  // 数据处理,将数据实时显示在container中
  const getDataHandler = () => {
    timer = null;
    let arrstr = arr.join('');
    arrstr = arrstr.replace(/\r/g,'')
    arr = [];
    message_container.innerText += arrstr;
    // avoid DOM thrashing
    scroll_height = message_container.scrollHeight
    scroll_width = message_container.scrollWidth
    if(print_num > 5000) {
      message_container.innerText = ''
      print_num = 0
    }
    if(need_scroll) {
      arrstr.includes('\n') ? message_container.scrollTop = scroll_height - height :
      message_container.scrollLeft = scroll_width - width;
    }
  }
  // 'data'事件响应方法
  const getData = data => {
    // 先存入数据
    arr.push(data);
    print_num += 1
    // 再在16ms延时之后更新数据
    !timer && (timer = setTimeout(getDataHandler, 16));
  }
  // 连接串口
  const connect= () => {
    clear()
    sp = new serialport.SerialPort(port, {baudRate: +select_rate.value});
    sp.on('data',getData);
  }
  // 取消连接
  const disconnect = () => {
    arr = [];
    sp.removeListener('data', getData);
    sp.once('close', () => {
      ipcRenderer.send('already_stopped');
      sp = null;
    })
    sp.close();
  }
  // 事件监听
  current_window.once('close',disconnect);
  ipcRenderer.on('new_window_create',(e, comm) => {
    port = comm;
    connect();
  })

  ipcRenderer.on('stop_now', disconnect);
  ipcRenderer.on('reconnect_now', connect);
  // 发送消息不能为空
  $.select('#send').onclick = () =>{
    const send_value = send.value;
    if (!send_value) {
      return dialog.showMessageBox(current_window,{type: 'info',detail: '没有消息可以发送',title: ' '});
    }
    sp.write(send_value + '\r\n');
    send.value = '';
  }
  // 波特率的改变
  select_rate.onchange = () => {
    // sp.removeListener('data', getData);
    sp.once('close', () => {
      connect();
    })
    sp.close();
  }
  // 是否启用滚屏
  $.select('.footer input')[0].onchange = (e) => {
    need_scroll = e.target.checked;
  }
  // 清屏
  $.select('#clear').onclick = clear;
  function clear() {
    message_container.innerText = '';
  }
}
```

```html
<body>
  <div class='modal_container'>
    <div class="header">
      <input type="text" id="send_value"/>
      <button type="text" id="send">发送</button>
    </div>
    <span class="message_container" readonly wrap="soft">
    </span>
    <div class="footer">
      <span id="roll"><input type="checkbox" checked/><span>自动滚屏</span></span>
      <span>波特率:
        <select id="rate_select">
          <option>300</option>
          <option>1200</option>
          <option>2400</option>
          <option>4800</option>
          <option selected>9600</option>
          <option>19200</option>
          <option>38400</option>
          <option>57600</option>
          <option>74880</option>
          <option>115200</option>
          <option>128000</option>
        </select>
      </span>
      <button id="clear">清空</button>
    </div>
    </div>
    <script src="./comm.js"></script>
</body>
```

## 界面展示:

![](http://p799phkik.bkt.clouddn.com/ExmHh3ytYj.gif)

## 总结:

现附上所有会导致layout thrashing的常见列表(其余还有SVG和webkit prefixed APIs待补充)

### elementl类:

- clientHeight\clientWidth\clientTop\clientLeft
- offsetHeight\offsetWidth\offsetLeft\offsetTop\offsetParent
- scrollHeight\scrollWidth\scrollTop\scrollLeft\scrollIntoView\scrollIntoViewIfNeeded\scrollByLines\scrollByPages
- getBoundingClientReact\getClientRects
- focus
- innerText

### MouseEvent类

- layerX\layerY\offsetX\offsetY

### window类

- getComputedStyle
- scrollBy\scrollTo\scrollX\scrollY

### Frame,Document,Image类

- height\width
