---
title: 笔记-ECMAScript 6.0
date: 2020-04-25 00:54:39
categories: 前端学习
tags:
- [前端笔记]
- [JavaScript]
- [ES6]
---

ES6的新特性简单介绍

<!-- more -->

# ES6

##0. 块级作用域

 ES6之前没有块级作用域，**ES5的var没有块级作用域**的概念，只有function有作用域的概念。ES6的let、const引入了块级作用域。

 **ES5之前if和for都没有作用域**，所以很多时候需要使用function的作用域，比如闭包。

### 0.1. 没有块级作用域出现的问题

####0.1.0. ES5的**if**没有块级作用域

下列代码输出结果为`'zzz','ttt','ttt'`，第一次调用func()，此时name=‘zzz’，在if块外将name置成‘ttt’，此时生效了，if没有块级作用域。

```javascript
var func;
if(true){
	var name = 'zzz';
	func = function (){
		console.log(name);
	}
	func();
}
name = 'ttt';
func();
console.log(name);
```

####0.1.1. ES5的for没有块级作用域

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>块级作用域</title>
</head>
<body>
  <button>按钮1</button>
  <button>按钮2</button>
  <button>按钮3</button>
  <button>按钮4</button>
  <button>按钮5</button>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js">    </script>
    <script>
      var btns = document.getElementsByTagName("button");
      for (var i = 0; i < btns.length; i++) {
        btns[i].addEventListener('click',function (param) {
        console.log("第"+i+"个按钮被点击了");
        });
      }
    </script>
</body>
</html>
```

 for块级中使用`var`声明变量i时，是全局变量，点击任意按钮结果都是“第五个按钮被点击了”。说明在执行`btns[i].addEventListener('click',function())`时，for块级循环已经走完，此时`i=5`，所有添加的事件的i都是5。

而使用闭包能解决上述问题

```javascript
for (var i = 0; i < btns.length; i++) {
	(function (i) {
		btns[i].addEventListener('click',function (param) {
			console.log("第"+i+"个按钮被点击了");
		})
	})(i);
}
```

或在ES6中使用let/const解决块级作用域问题，let和const有块级作用域，const定义常量，在for块级中使用let解决块级作用域问题。

```javascript
// ES6使用let/const
const btns = document.getElementsByTagName("button");
	for (let i = 0; i < btns.length; i++) {
	btns[i].addEventListener('click',function (param) {
		console.log("第"+i+"个按钮被点击了");
	})
}
```

####0.1.2. const的使用

- const用来定义常量，赋值之后不能再赋值，再次赋值会报错。

- const不能只声明不赋值，会报错。

- const常量含义是你不能改变其指向的对象，例如user，但是你可以改变user属性。

  ```javascript
      <script>
          const user = {
              name:"zzz",
              age:24,
              height:175
          }
          console.log(user)
          user.name = "ttt"
          user.age = 22
          user.height = 188
          console.log(user)
      </script>
  ```

##1. ES6的增强写法

###1.0. ES6的对象属性增强型写法

 ES6以前定义一个对象

```javascript
const name = "zzz";
const age = 18;
const user = {
  name:name,
  age:age
}
console.log(user);
```

 ES6写法

```javascript
const name = "zzz";
const age = 18;
const user = {
	name,age
}
console.log(user);
```

###1.1. ES6对象的函数增强型写法

 ES6之前对象内定义函数

```javascript
const obj = {
  run:function(){
     console.log("奔跑");
  }
}
```

ES6写法

```javascript
const obj = {
  run(){
     console.log("奔跑");
  }
}
```

```javascript
// 在外部定义函数
function success() {
    console.log("success")
}
// 在对象内部只用 名字 进行调用
const obj = {
  success
}
```

##2. 箭头函数

###2.0. 认识箭头函数

> 传统定义函数的方式

```javascript
const aaa = function (param) {
    
}
```

> 对象字面量中定义函数

```javascript
const obj = {
    bbb (param) {}
}
```

> ES6中的箭头函数

```javascript
// const ccc = (参数列表) =>{}
const ccc = () => {}
```

###2.1. 箭头函数的参数和返回值

####2.1.0. 参数问题

> 放两个参数

```javascript
const sum = (num1,num2) => {
    return num1 + num2
}
```

> 放入一个参数时，()可以省略

```javascript
const power = num => {
    return num * num
}
```

####2.1.1. 函数内部

> 函数内部代码块中有多行代码时

```javascript
const test = () => {
    console.log("hello zzz")
    console.log("hello vue")
}
```

> 代码块中只有一行代码，可以省略**{}**和return

```javascript
// const mul = (num1,num2) => {
//   return num1 * num2
// }
const mul = (num1,num2) => num1* num2
// const log = () => {
//   console.log("log")
// }
const log = () => console.log("log")
```

####2.1.2. 箭头函数this的使用

##### 什么时候使用箭头函数

- 对于需要使用object.method()方式调用的函数，使用普通函数定义，不要使用箭头函数。对象方法中所使用的this值有确定的含义，指的就是object本身。
- 其他情况下，全部使用箭头函数。

> 箭头函数**没有自己的this值**，箭头函数中所使用的this都是来自函数作用域链，它的取值遵循普通变量一样的规则，在函数作用域链中**一层一层往上找**。

##### 测试示例

```javascript
    const obj = {
      aaa() {
        setTimeout(function () {
            
          setTimeout(function () {
            console.log(this) //window
          })
            
          setTimeout(() => {
            console.log(this) //window
          })
            
        })
          
        setTimeout(() => {
            
          setTimeout(function () {
            console.log(this) //window
          })
            
          setTimeout(() => {
            console.log(this) //obj
          })
            
        })
          
      }
    }
    obj.aaa()
```

##3. 高阶函数

###3.0. filter过滤函数

```javascript
const nums = [2,3,5,1,77,55,100,200]
//要求：获取nums中大于50的数
//回调函数会遍历nums中每一个数，传入回调函数，在回调函数中写判断逻辑，返回true则会被数组接收，false会被拒绝
let newNums = nums.filter(function (num) {
  if(num > 50){
    return true;
  }
  return false;
 })

//可以使用箭头函数简写
//  let newNums = nums.filter(num => num >50)
```

###3.1. map高阶函数

```javascript
// 要求：将上面已经过滤的新数组每项乘以2

//map函数同样会遍历数组每一项，传入回调函数为参数，num是map遍历的每一项，回调函数function返回值会被添加到新数组中
let newNums2 = newNums.map(function (num) {
  return num * 2
 })

//箭头函数简写
//  let newNums2 = newNums.map(num => num * 2)
console.log(newNums2);
```

###3.2. reduce高阶函数

```javascript
//要求：将上面map函数得到的newNums2数组所有数累加

//reduce函数同样会遍历数组每一项，传入回调函数和‘0’为参数，0表示回调函数中preValue初始值为0，回调函数中参数preValue是每一次回调函数function返回的值，currentValue是当前值
//例如数组为[154, 110, 200, 400],则回调函数第一次返回值为0+154=154，第二次preValue为154，返回值为154+110=264，以此类推直到遍历完成
let newNum = newNums2.reduce(function (preValue,currentValue) {
  return preValue + currentValue
 },0)

//箭头函数简写
// let newNum = newNums2.reduce((preValue,currentValue) => preValue + currentValue)
console.log(newNum);
```

###3.3. filter、map和reduce三者结合使用

```javascript
//上述三个需求综合使用
let n = nums.filter(num => num > 50).map(num => num * 2).reduce((preValue,currentValue) => preValue + currentValue)
console.log(n);
```

##4.  参数解构

**解构**(Destructuring)：是将一个数据结构分解为更小的部分的过程。大大的简化数组或者对象里面的元素的赋值语句。
在ES6中，从数组和对象中提取值，对变量进行赋值。

###4.0. 数组的解构赋值

> 使用ES6的结构语法，可以一次性给多个变量赋值。还有可以很简单的交换变量值：

####4.0.0. ES6前的数组赋值方式

```javascript
var arr = [1,2,3];
console.log(arr[0]); //1
console.log(arr[1]); //2
```

####4.0.1. ES6的数组赋值方式一

```javascript
var [a,b,c] = [1,2,3];
console.log(a); //1
console.log(b); //2
```

####4.0.2. ES6的数组赋值方式二

```javascript
var [a,[b,c]] = [1,[2,3]];
console.log(a);//1
console.log(b);//2
console.log(c);//3
```

####4.0.3. ES6的数组赋值方式三

```javascript
var [a,b,c] = new Set([4,5,6]);
console.log(a);//4
console.log(b);//5
console.log(c);//6
```

####4.0.4. ES6的数组赋值方式四

```javascript
var [a=1,b=a] = [];// 相当于指定了a和b的默认值，a=1,b=a=1
console.log(a);//1
console.log(b);//1
```

####4.0.5. ES6的数组赋值方式五

```javascript
var [a=1,b=a] = [2,3];
console.log(a);//2
console.log(b);//3
```

####4.0.6. ES6的数组赋值方式六

```javascript
//类似函数的剩余参数，数组解构有个类似的，剩余项的概念。 ...tail，里面的tail就是剩余项，也是个数组
let [head,...tail] = [1,2,3,4]; //head:1,tail:[2,3,4]
```

####4.0.7. ES6中用剩余项克隆数组的方式

```javascript
let colors = ["red","green","blue"];
let [...cloneColors] = colors;
// 需注意，剩余项...cloneColors是最后一项，后面不能有逗号
console.log(cloneColors)  //["red", "green", "blue"]
```

##### 若只想获取数组中的第三个元素，则不必给前两项提供变量名。

```javascript
let colors = ["red","green","blue"];
let [,,thirdColor] = colors;
console.log(thirdColor)  //blue
```

###5.0. 对象的解构赋值

####5.0.0. ES6前的对象赋值

```javascript
var json = {
    a:1,
    b:2 
};
```

####5.0.1. ES6的对象赋值

```javascript
var {a:name,b} = {a:3,b:4};
var {a,b} = {a:3,b:4};
var {a,b} = {a:1,b:[2,3]};
```

例：

```javascript
var json = {
	a:[
		'hi',
		{b:'hello'}
	]
};

var {a:[name,{b}]} = json;
console.log(name);//hi
console.log(b);//hello
```

###6.0. 函数的解构赋值

####6.0.0. ES6前的函数赋值

```javascript
　　function fn(a,b){
　　　　console.log([a,b]);
　　}
　　fn(1,2);//[1,2]
```

####6.0.1. ES6的函数赋值方式一

```javascript
function fn([a,b]){
	console.log([a,b]);
}
fn([3,4]);//[3, 4]
```

####6.0.2. ES6的函数赋值方式二

```javascript
　　function fn({a=1,b=2} = {}){
　　　　console.log([a,b]);
　　}

　　fn(); //[1, 2]
　　fn({a:1}); //[1, 2]
　　fn({a:10,b:11}); //[10, 11]
```

##### ES6的函数赋值方式三

```javascript
　　function fn({a,b} = {a:1,b:2}){
　　　　console.log([a,b]);
　　}
　　fn(); //[1, 2]
　　fn({a:1}); //[1, undefined]
　　fn({a:10,b:11}); //[10, 11]
```

##7.0. ES6的字符串拼接

> 不需要任何的加号和引号，全部字符仅仅由一组``符号包裹即可，而放置动态数据或者变量即用${变量}方式即可。

```javascript
var firstname="张"
var lastname="三"
var newSplicing=`我姓${firstname}名${lastname}`
console.log(newSplicing)  //我姓张名三
```

