---
layout:       post
title:        "第一次转岗面试"
subtitle:     "毫无准备的技术面"
date:         2018-04-23 12:00:00
author:       "Lorry"
header-mask:  0.3
header-img: 'http://p799phkik.bkt.clouddn.com/paper.jpeg'
catalog:      true
multilingual: false
tags:
    - interview
---
# 人生总是需要那么一点出乎意料

上周五(04-20)刚跟HR说了渴望转岗前端开发的事情,让HR帮忙给问问流程,结果今天下午,猛不丁的豆哥(技术部的招聘负责人)就过来了.猝不及防,一点准备都没有,也不能不上啊..所以硬着头皮过去"聊聊"

全称紧张,之前记得的好多东西都想不起来了.很多问题没答上来,自己也很不满意今天的表现,必须得认认真真记录一下自己的处女技术面,引以为戒!也留存记忆.

1. 简单说说在做过的项目

> 1 在公司使用egret引擎做游戏前端开发,完成了代码竞技场的游戏UI前端实现.2 "通天塔"项目的题库后台新需求开发.多是直接沿用之前写好的类和接口,没有太深入的使用.

2. 在egret开发过程中遇到的最大的问题是什么?怎么解决的.
> (当时是遇到了好多的问题,现在要让我想一个具体的一时间没想起来)我回答了之前折腾我很久的图片资源的加载问题.这里回答十分的没逻辑性,实际上是白鹭自动布局中的[失效验证机制](http://developer.egret.com/cn/github/egret-docs/extension/EUI/autoLayout/FailureToVerify/index.html),当加载子组件到父容器当中,如果子组件尺寸的改变,在这个改变的这一帧里的height和width还没有实质的发生更改.所以是不能立即获取
到高宽的.有两种方法可以解决这个问题,一个是使用callLater,在本帧的失效验证之后再获取.2是使用validateNow来立即生效.

3. 对react有什么理解,觉得跟原生HTML有什么优势?
> 1 react有更丰富的第三方包,可以省去很多造轮子的工作,操作DOM的方法也更多.
> 2 react更方便打包和版本管理.
> 3 组件化模块化开发.
4. ES6有什么新特性
> promise
> let const
> 箭头函数
> 字符串模板
> class语法糖
> __proto__的支持
> import和export
> default parameter
> 解构
- 数组[]和对象{}的赋值
- ...的不定参数应用
- MAP 的 for of 应用
- 模块加载 import {} from xxx const {xxx} =require('xxx')

5. 对Promise有什么了解?then里面只能有一个参数吗?
> 三种状态 1. pending 2. rejected 3. resolved.
> resolve() reject()对应then(),catch()
> then中可以有两个参数then(resolve(),reject()),第一个函数处理完成,第二个处理拒绝.其实这里豆哥以及提示我了,除了catch之外还有别的catch方法吗?我没想起来,太紧张了..

6. 闭包有什么作用?
> 创建块级作用域,避免全局变量的污染(jQuery中的$符)
> 创建私有变量,私有方法

7. css如何实现垂直居中?
> top=50%,margin-top = -该元素宽度/2
> flex布局 align-items:center

8. 盒模型,如何确定使用哪一种模型?
> 2种,IE content + padding + border = width/height, 非IE content = width/height,
> 居然没回答上来如何确定,box-sizing啊!!使用border-box还是content-box

9. prototype是什么东西?
> 原型链,用于定义函数方法的,所有的函数都是对象,prototype最终都指向null
> 用于继承

10. 如何将现有代码竞技场项目迁移到react中?觉得白鹭跟react有什么区别?
> 1 模块依赖和引入的修改,这个问题还不是很懂,所以也没回答好.

11. 使用的什么系统和编译器?
> windows,编译器egret用的wing,平时用的vs code,早晚要换到Ubantu,已经发现部分的react第三方包对windows有歧视...

## 在路上,一切都是脚下的开始!加油!
