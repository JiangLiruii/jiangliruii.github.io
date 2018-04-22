---
layout:     post
title:      "jQuery"
subtitle:   " 之:this使用方法"
date:       2018-03-13 12:00:00
author:     "Lorry"
catalog: true
multilingual: false
header-img: "http://p799phkik.bkt.clouddn.com/micro_building.jpeg"
tags:
    - jQuery
---
# 事件代理
在jQuery使用on方法进行事件代理的时候,this是有多种变化的.this对js的dom操作至关重要,下面让我们开始对this进行研究.

> 放一段示例的HTML的代码:

``` html
<ul id="selected-plays" class="clear-after">
        <li>Comedies
          <ul>
            <li><a href="/asyoulikeit/">As You Like It</a></li>
            <li>All's Well That Ends Well</li>
            <li>A Midsummer Night's Dream</li>
            <li>Twelfth Night</li>
          </ul>
        </li>
        <li>Tragedies
          <ul>
            <li><a href="hamlet.pdf">Hamlet</a></li>
            <li>Macbeth</li>
            <li>Romeo and Juliet</li>
          </ul>
        </li>
        <li>Histories
          <ul>
            <li>Henry IV (<a href="mailto:henryiv@king.co.uk">email</a>)
              <ul>
                <li>Part I</li>
                <li>Part II</li>
              </ul>
            </li>
            <li><a href="http://www.shakespeare.co.uk/henryv.htm">Henry V</a></li>
            <li>Richard II</li>
          </ul>
        </li>
      </ul>

```
> 显示效果为

***
<ul id="selected-plays" class="clear-after">
        <li>Comedies
          <ul>
            <li><a href="/asyoulikeit/">As You Like It</a></li>
            <li>All's Well That Ends Well</li>
            <li>A Midsummer Night's Dream</li>
            <li>Twelfth Night</li>
          </ul>
        </li>
        <li>Tragedies
          <ul>
            <li><a href="hamlet.pdf">Hamlet</a></li>
            <li>Macbeth</li>
            <li>Romeo and Juliet</li>
          </ul>
        </li>
        <li>Histories
          <ul>
            <li>Henry IV (<a href="mailto:henryiv@king.co.uk">email</a>)
              <ul>
                <li>Part I</li>
                <li>Part II</li>
              </ul>
            </li>
            <li><a href="http://www.shakespeare.co.uk/henryv.htm">Henry V</a></li>
            <li>Richard II</li>
          </ul>
        </li>
</ul>
***

# 使用不同方式对事件绑定
1. **如果on函数中没有没有第二个参数,且绑定的元素唯一,则this指向on前绑定的元素**
```javascript
$('#selected-plays').on('click',function(e){
    console.log(this);
})
```
2. **如果绑定的元素不唯一,比如一下代码有多个li匹配元素,所以当任意一个li元素点击的时候会向上冒泡,直达最外层,比如此时点击**
```javascript
$('#selected-plays li').on('click',function(e){
    console.log(this);
})
```
![效果](http://p799phkik.bkt.clouddn.com/domlist.png)
点击事件会触发以下元素响应(这是jQuery的遍历机制).
![](http://p799phkik.bkt.clouddn.com/domlist2.png)
使用禁止冒泡:
```javascript
$('#selected-plays li').on('click',function(e){
    console.log(this);
    e.stopPropagation();})
```
此时this就只有最底层的li响应了.
3. **如果on中有第二个参数:**
```javascript
$('#selected-plays').on('click','li',function(e){
    console.log(this);})
```
此处结果跟在on前绑定的结果一样,只不过返回的值不一样(返回值为on前绑定的jQuery对象)
```javascript
$('#selected-plays li').on('click',function(e){})
```
同样的,如果第二个参数唯一的话,也等同于on前绑定唯一的jQuery对象
4. **还有一个很重要的情况,即*绑定在document上*,这也是事件代理中最常用的形式.**
```javascript
$(document).on('click','li',function(e){
    console.log(this);
})
```
同样的,会冒泡直到顶层document,同样可以使用stopPropagation()
```javascript
$(document).on('click','li',function(e){
    console.log(this);
    e.stopPropagation();})
```
![](https://images2018.cnblogs.com/blog/1215846/201803/1215846-20180313173215110-1345153175.png)

## 总结:click会由当前点击元素最近的匹配元素一直冒泡到document中