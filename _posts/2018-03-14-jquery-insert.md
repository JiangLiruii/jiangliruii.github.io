---
layout:     post
title:      "jQuery"
subtitle:   "之:插入方式"
date:       2018-03-14 12:00:00
author:     "Lorry"
catalog: true
multilingual: false
tags:
    - jQuery
---
# jQuery中append appendTo prepend prependTo insertBefore insertAfter after before之间的区别

## 使用方法都很类似,但是呈现的结果有所不同.在不同的场景下有选择性的使用才能达到我们预期的效果.

> 先给一段代码,用于演示不同的插入有什么不同的结果

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>test</title>
    <script src="jquery.js"></script>
  </head>
  <style>
  p {
    background-color: aquamarine;
  }
  </style>
  <body>
    <p id="n1">
      <span id="n2">span#n2</span>n1    
  </p>
  <p id="n3">
      <label id="n4" class="move">label#n4</label>n3
  </p>
  <p id="n5">
      <span id="n6">span#n6</span>n5
  </p>
  <p id="n7">
    <span id="n8">span#n8</span>n7
  </p>
  <p id="n9">
    <span id="n10">span#n10</span>n9
  </p>
  <p id="n11">
    <span id="n12">span#n12</span>n11
  </p>
  </body>
</html>
```
先新建一个jQuery div对象:
``` javascript
let newdiv = $('<div>new one</div>');
```
分别使用在console中执行下列语句看看有什么效果
``` javascript
$('#n1').append(newdiv.clone());
newdiv.clone().insertAfter('#n1');
newdiv.clone().appendTo('#n3');
$('#n5').prepend(newdiv.clone());
$('#n5').insertBefore(newdiv.clone());
newdiv.clone().prependTo('#n7')
$('#n9').before(newdiv.clone());
newdiv.clone().before('#n9');
$('#n11').after(newdiv.clone());
newdiv.clone().after('#n11');
```
具体结果就不再展示,最后的总结:

1 To和不带To就是一个相反的关系

2 **pend是在p元素内末尾添加,成为p的last-child insert**是在p元素外添加,成为p的next,或first-sibling

3 A.before(B) = B.insertBefore(A) A.after(B) = B.insertAfter(A)效果上是相等的,但是返回值不一样,如果使用before返回的是A,如果是insert返回的是B

