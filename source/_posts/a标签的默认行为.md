---
title: a标签的默认行为
date: 2020-05-22 20:23:00
categories: 前端学习
tags:
- [a标签]
- [小细节]
---

a标签的默认行为以及修改、阻止的方式。相关笔记文字描述来自[lincimy的简书](https://www.jianshu.com/p/8a2bd9792eec)。

<!-- more -->

# a标签的默认行为

## 概念

a标签的默认跳转链接行为是由`href`属性来实现的，同时设置`href`属性可以使a标签在hover状态下以手指指示的样式显示。但实际过程中发现**对a标签的href属性的不同设置，可能会导致不同的行为反馈**。

## 1.a标签中设置href属性，没有赋任何值

前端代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>a标签的默认行为</title>
    <script src="vue.js"></script>
</head>
<body>
    <div id="app">
        <h3>我是文字</h3>// 此处略去剩余的29行重复标签
        <a href="" @click="print">a标签，href属性为空</a><br/>
    </div>

    <script>
        let vue = new Vue ({
            el: "#app",
            methods: {
                print () {
                    console.log("一闪而过的弹窗");
                }
            }
        })
    </script>
</body>
</html>
```

情况演示

![a标签的默认行为一](/a标签的默认行为/a标签的默认行为一.gif)

可以看出，当href属性为空时，点击a标签会刷新页面，回到顶部。

## 2.a标签中设置href属性，赋值href="#"

当把上面的a标签的href属性改为`#`时，会出现如下情况

![a标签的默认行为一](/a标签的默认行为/a标签的默认行为二.gif)

点击a标签后会回到页面顶部，但不刷新页面。

## 修改a标签的默认行为

- 使用`javascript:void(0)`给href属性赋值
  - void 是JavaScript 的一个运算符，void(0)就是什么都不做的意思，点击之后也不会回到页面顶部，使用javascript代码阻止了href属性的默认跳转链接行为。a标签点击后会执行@click中设定函数print()。

```javascript
 <a href="javascript:void(0)" @click="print">a标签， 使用javascript语句给href属性赋值</a>
```

- 使用`javascript:;`给href赋值
  - javascript: 是一个伪协议，其他的伪协议还有 mail:  tel:  file:  等等。
  - javascript:是表示在触发\<a>默认动作时，执行一段JavaScript代码，而` javascript:;` 表示什么都不执行，这样点击\<a>时就没有任何反应。

```javascript
<a href="javascript:;" @click="print">用javascript语句给href属性赋值</a>
```

![用javascript语句给href属性赋值](/a标签的默认行为/用javascript语句给href属性赋值.gif)

## 阻止a标签的默认行为

- 让 a 标签的点击事件的处理函数的返回值为 false ，这样 a 链接就不会跳转。

```javascript
<div>        
	<a href="http://www.baidu.com">原始DOM操作</a><br/>
</div>
<script>
	document.querySelector("a").onclick = function () {
        console.log("弹一弹");
        return false;
    }
</script>
```

- 使用e.preventDefault()

```javascript
<div>        
	<a href="http://www.baidu.com">原始DOM操作</a><br/>
</div>
<script>
	document.querySelector("a").onclick = function (e) {
        console.log("弹一弹");
        e.preventDefault();
    }
</script>
```

注：上面的两种操作都是在**原生JS**里使用的

![用javascript语句给href属性赋值](/a标签的默认行为/阻止a标签的默认行为.gif)









