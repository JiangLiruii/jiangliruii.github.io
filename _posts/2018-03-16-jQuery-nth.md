---
layout:     post
title:      "jQuery"
subtitle:   " 之:nth伪类选择器"
date:       2018-03-16 12:00:00
author:     "Lorry"
catalog: true
multilingual: false
tags:
    - jQuery
---
# 说在前面
## 在jQuery中有不少的伪类选择器,给了我们在选取元素时很大的帮助,可以减少类和id的定义.下面时对:nth选择器的总结和分析.
> 来贴一段代码,具体效果可[点击](../_data/2018-03-16-jQuery-nth.html)
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script src="http://cdn.static.runoob.com/libs/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
  $("button").click(function(){
    var btn = $(this).text();
    $("p").css("background-color","white"); 
    $("p" + btn).css("background-color","yellow");  
  });
});
</script>

</head>
<body>
<button>:nth-child(2)</button>
<button>:nth-last-child(2)</button>
<button>:nth-of-type(2)</button>
<button>:nth-last-of-type(2)</button>

<h1>body 中的标题</h1>
<p>body 中第一个段落。</p>
<p>body 中第二个段落。</p>

<div style="border:1px solid;">
    <span>div 中的 span 元素</span>
    <p>div 中的第一个段落。</p>
    <p>div 中的第二个段落。</p>
    <p>div 中的第三个段落。</p>
    <p>div 中的第四个段落。</p>
    <span>div 中的 span 元素</span>
</div><br>

<div style="border:1px solid;">
    <p>另一个 div 中的第一个段落。</p>
    <p>另一个 div 中的第二个段落。</p>
    <p>另一个 div 中的最后一个段落。</p>
</div>

<p>body 中最后一个段落。</p>

</body>
</html>
```
> 整个页面的显示结果为:
> ![显示结果](https://images2018.cnblogs.com/blog/1215846/201803/1215846-20180316155713538-118512035.png)

### 那么**从左往右**会点击button, 哪几行会出现颜色的变化呢?先对这四种类型进行分析
|标签名|解释
|:-:|:-:|
|p:nth-child(n)|p标签的父元素的第n个标签|
|p:nth-last-child(n)|p标签的父元素的倒数第n个标签|
|p:nth-type-of(n)|p标签的父元素的第n个p标签|
|p:nth-last-type-of(n)|p标签的父元素的倒数第n个p标签|

> 现在可以根据上面的表格做出判断了
|点击button|变化行数
|:-:|:-:|
|1|4 10|
|2|7 10|
|3|2 5 10|
|4|2 6 10|
***
###  由此可以看出 
A:nth-child(B)表示在父元素的第B个元素刚好是A类型的所有A元素

A:nth-of-type(B) 表示父元素的第B个A元素的所有A元素集合

看出来区别了吗?nth-of-type实质上在索引时进行了一次筛选,父元素只包含A,所以只要B < A在父元素中的数量就一定有值,而nth-child没有进行筛选,是父元素中所有元素的第B个,如果是A则取出来, 有可能就取不到.比如第一次取在body下的p:nth-child(2)是button标签,不是p标签,不会选择出来.

### 如果你能理解以上的问题,那么对eq()的区别也显而易见了,A:eq(B),选出的所有A中的第B个,跟父元素就没关系了.

