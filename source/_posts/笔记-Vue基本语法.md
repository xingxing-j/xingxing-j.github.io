---
title: 笔记-Vue基本语法
date: 2020-04-24 12:50:32
categories: 前端学习
tags:
- [前端学习]
- [Vue]
- [前端框架]
---

Vue基础语法的使用了解

<!-- more -->

# Vue基本语法

## 简述

**Vue.js**也称为Vue，读音/vju:/，类似view。

- 是一个构建用户界面的框架
- 是一个轻量级MVVM（Model-View-ViewModel）框架
- **数据驱动**+**组件化**的前端开发（核心思想）
- 通过简单的API实现**响应式的数据绑定**和**组合的视图组件**
- 本身只关注UI，可轻松引入vue插件和其他第三方库开发项目 

## 命令式编程和指令式编程

### 命令式编程

 原生js做法

1. 创建div元素，设置id属性
2. 定义一个变量叫message
3. 将message变量放在div元素中显示
4. 修改message数据
5. 将修改的元素替换到div

### 声明式编程

 vue写法

1. 创建一个div元素，设置id属性
2. **定义一个vue对象，将div挂载在vue对象上**
3. 在vue对象内定义变量message，并**绑定数据**
4. 将message变量放在div元素上显示
5. 修改vue对象中的变量message，div元素数据自动改变

语法注意：`new Vue`时，**Vue需首字母大写**

语法

```javascript
var vm = new Vue({
  // 选项
})
```

示例

```html
<div id="vue_det">
    <h1>site : {{site}}</h1>
    <h1>url : {{url}}</h1>
    <h1>{{details()}}</h1>
</div>
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue_det',// 用于要挂载管理的元素，'vue_detDOM'为元素中的 id
        data: {// 用来定义属性
            site: "菜鸟教程",
            url: "www.runoob.com",
            alexa: "10000"
        },
        methods: {// 用来定义函数，可以通过 return 来返回函数值。
            details: function() {
                return  this.site + " - 学的不仅是技术，更是梦想！";
            }
        }
    })
</script>
```

## Vue相关插件

- **vue-cli**:vue 脚手架 
- **vue-resource(axios)**:ajax 请求 
- **vue-router**: 路由 
- **vuex**: 状态管理 
- **vue-lazyload**: 图片懒加载 
- **vue-scroller**: 页面滑动相关 
- **mint-ui**: 基于 vue 的 UI 组件库(移动端)
- **element-ui**: 基于 vue 的 UI 组件库(PC 端)

## 模板语法

### 模板的理解

动态的html页面，包含了一些JS语法代码。例如：Mustache语法和指令(以v开头的标签属性)

### 插值操作

#### Mustache语法

mustache是胡须的意思，因为{&#123; &#125;}像胡须，又叫大括号语法。

在vue对象挂载的dom元素中，{&#123; &#125;}不仅可以**直接写变量**，还可以**写简单表达式**，并且对标签不敏感。

##### 示例

```html
<div id="app">
    <h2>{{message}}</h2>
    <h2>{{message}},啧啧啧</h2>

    <!-- Mustache的语法不仅可以直接写变量，还可以写简单表达式，函数，三目运算符... -->
    <h2>{{firstName + lastName}}</h2>
    <h2>{{firstName + " " + lastName}}</h2>
    <h2>{{firstName}}{{lastName}}</h2>
    <h2>{{count * 2}}</h2>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        message:"你好啊",
        firstName:"aaa",
        lastName:"bbb",
        count:100
      }
    })
  </script>
```

#### v-html

在某些时候我们不希望直接输出`<a href='http://www.baidu.com'>百度一下</a>`这样的字符串，而输出被html自己转化的超链接。此时可以使用**v-html**。

```html
  <div id="app">
    <h2>不使用v-html</h2>
    <h2>{{url}}</h2>
    <h2>使用v-html，直接插入html</h2>
    <h2 v-html="url"></h2>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        message:"你好啊",
        url:"<a href='http://www.baidu.com'>百度一下</a>"
      }
    })
  </script>
```

#### v-text

v-text会覆盖DOM元素中的数据，相当于JS的**innerHtml**方法

使用{&#123;message&#125;}是拼接变量和字符串，而是用v-text是**直接覆盖字符串内容**。

```html
  <div id="app">
    <h2>不使用v-text</h2>
    <h2>{{message}}，啧啧啧</h2>
    <h2>使用v-text，以文本形式显示,会覆盖下面的 "，啧啧啧" </h2>
    <h2 v-text="message">，啧啧啧</h2>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        message:"你好啊"
      }
    })
  </script>
```

#### v-pre

有时候我们期望直接输出{&#123;message&#125;}这样的字符串，而不是被{&#123;&#125;}语法转化的message的变量值，此时可以使用`v-pre`标签。

使用v-pre修饰的dom会直接输出字符串。

```html
  <div id="app">
    <h2>不使用v-pre</h2>
    <h2>{{message}}</h2>
    <h2>使用v-pre,不会解析</h2>
    <h2 v-pre>{{message}}</h2>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        message:"你好啊"
      }
    })
  </script>
```

#### v-cloak

有时因为加载延时问题，数据没有及时刷新，就造成了页面显示从{&#123;message&#125;}到message变量“你好啊”的变化，这样闪动的变化，会造成用户体验不好。此时需要使用到`v-cloak`的这个标签。可与 css 配合: [v-cloak] { display: none }
在vue**解析之前**，div属性中有`v-cloak`这个标签，在vue**解析完成之后**，v-cloak标签被移除。
类似于div开始有一个css属性`display:none;`，加载完成之后，css属性变成`display:block`，元素才显示出来。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>v-cloak指令的使用</title>
  <style>
    [v-cloak]{
      display: none;
    }
  </style>
</head>

<body>
  <div id="app" v-cloak>
    <h2>{{message}}</h2>
  </div>
    
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    //在vue解析前，div中有一个属性v-cloak
    //在vue解析之后，div中没有一个属性v-cloak
    setTimeout(() => {
      const app = new Vue({
        el: "#app",
        data: {
          message: "你好啊"
        }
      })
    }, 1000);
  </script>
</body>
</html>
```

### v-bind

作用：**动态绑定语法属性**

#### 写法

##### 基本写法

```html
<img v-bind:src="imgUrl"/>
```

##### 语法糖写法

```html
<!-- 即v-bind：的写法可省略成 : -->
<img :src="imgUrl"/>
```

##### v-bind动态绑定class

- 绑定方式：对象语法。对象语法的含义是**class后面跟的是一个对象**。

```html
<!-- 对象里冒号左边是类选择器的名字，右边是data里的具体属性名，且右边的值需为boolean类型-->
<!-- 用法一:直接通过绑定一个类 -->
<h2 :class-"{active: isActive}">Hello World /h2>

<!-- 用法二:也可以通过判断，传入多个值 -->
<h2 :class="{active: isActive,line: isLine}">Hello World/h2>

<!-- 用法三:和普通的类同时存在，并不冲突。注:如果isActive和isLine都为true，那么会有tit1e/active/line三个类 -->
<h2 class="title" :class="[active： isActive, line： istine]">Hello world</h2>

<!-- 用法四:如果过于复杂，可以放在一个methods或者.computed中注:classes是一个计算属性 -->
<h2 class="title" :class="classes">Hello bor ld</h2>
```
- 绑定方式：数组语法。数组语法的含义是**class后面跟的是一个数组**。

```html
<!-- 数组里的各项值是data里的属性名，且对应着类选择器的名字-->
<!-- 用法一:直接通过绑定一个类 -->
<h2 :class-"[active]">Hello World /h2>

<!-- 用法二:也可以直接传入多个值 -->
<h2 :class="[active, line]">Hello World/h2>

<!-- 用法三:和普通的类同时存在，并不冲突。注:会有tit1e/active/line三个类 -->
<h2 class="title" :class="[active, line]">Hello world</h2>

<!-- 用法四:如果过于复杂，可以放在一个methods或者.computed中注:classes是一个计算属性 -->
<h2 class="title" :class="classes">Hello bor ld</h2>
```

##### v-bind动态绑定class

- 绑定方式：对象语法。

```html
  <style>
    .title {
      font-size: 50px;
      color: red;
    }
  </style>
</head>
<body>
<!-- 写CSS的属性名的时候，可以使用驼峰式(fontSize)或短横线分隔，且用单引号括起来('font-size') -->
<div id="app">
  <!--<h2 :style="{key(属性名): value(属性值)}">{{message}}</h2>-->

  <!--'50px'必须加上单引号, 否则是当做一个变量去解析-->
  <!--<h2 :style="{fontSize: '50px'}">{{message}}</h2>-->

  <!--finalSize当成一个变量使用-->
  <!--<h2 :style="{fontSize: finalSize}">{{message}}</h2>-->
  <h2 :style="{fontSize: finalSize + 'px', backgroundColor: finalColor}">{{message}}</h2>
  <h2 :style="getStyles()">{{message}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      finalSize: 100,
      finalColor: 'red',
    },
    methods: {
      getStyles: function () {
        return {fontSize: this.finalSize + 'px', backgroundColor: this.finalColor}
      }
    }
  })
</script>

</body>
```

- 绑定方式：数组语法。

```html
<div id="app">
  <h2 :style="[baseStyle, baseStyle1]">{{message}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      baseStyle: {backgroundColor: 'red'},
      baseStyle1: {fontSize: '100px'},
    }
  })
</script>
```

### v-model

**Vue中使用v-model指令来实现表单元素和数据的双向绑定**。当然，我们也可以将v-model用于textarea元素。

```html
<div id="app">
  <input type="text" v-model="message">
  {{message}}
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    }
  })
</script>
```

#### v-model原理

`v-mode`其实是一个语法糖，它的背后本质上是包含两个操作：

1. `v-bind`绑定一个value属性
2. `v-on`指令给当前元素绑定input事件

```html
<div id="app">
  <!--<input type="text" v-model="message"> 等同于 
	  <input type="text" :value="message" @input="valueChange">-->
  <input type="text" :value="message" @input="message = $event.target.value">
  <h2>{{message}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    methods: {
      valueChange(event) {
        this.message = event.target.value;
      }
    }
  })
</script>
```

#### v-model结合radio

```html
<div id="app">
  <label for="male">
    <input type="radio" id="male" value="男" v-model="sex">男
  </label>
  <label for="female">
    <input type="radio" id="female" value="女" v-model="sex">女
  </label>
  <h2>您选择的性别是: {{sex}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      sex: '女'
    }
  })
</script>
```

#### v-model结合checkbox

复选框分为两种情况：单个勾选框和多个勾选框

- 单个勾选框，v-model即为布尔值。此时input的value并不影响v-model的值。
- 多个复选框
  - 当是多个复选框时，因为可以选中多个，所以对应的data中的属性是一个数组
  - 当选中某一个时，就会将input的value添加到数组中

```html
<div id="app">
 <h2>您的爱好是: {{hobbies}}</h2>

  <label v-for="item in originHobbies" :for="item">
    <input type="checkbox" :value="item" :id="item" v-model="hobbies">{{item}}
  </label>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      isAgree: false, // 单选框
      hobbies: [], // 多选框,
      originHobbies: ['篮球', '足球', '乒乓球', '羽毛球', '台球', '高尔夫球']
    }
  })
</script>
```

#### model结合select

和checkbox一样，select也分单选和多选两种情况。

- 单选
  - 只能选中一个值。v-model绑定的是一个值
  - 当我们选中option中的一个时，会将它对应的value赋值到mySelect中
- 多选：可以选中多个值
  - v-model绑定的是一个数组
  - 当选中多个值时，就会将选中的option对应的value添加到数组mySelects中

```html
<div id="app">
  <!--1.选择一个-->
  <select name="abc" v-model="fruit">
    <option value="苹果">苹果</option>
    <option value="香蕉">香蕉</option>
    <option value="榴莲">榴莲</option>
    <option value="葡萄">葡萄</option>
  </select>
  <h2>您选择的水果是: {{fruit}}</h2>

  <!--2.选择多个-->
  <select name="abc" v-model="fruits" multiple>
    <option value="苹果">苹果</option>
    <option value="香蕉">香蕉</option>
    <option value="榴莲">榴莲</option>
    <option value="葡萄">葡萄</option>
  </select>
  <h2>您选择的水果是: {{fruits}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      fruit: '香蕉',
      fruits: []
    }
  })
</script>
```

#### v-model修饰符

##### lazy修饰符

默认情况下，`v-model`默认是在input事件中同步输入框的数据的。也就是说，一旦有数据发生改变对应的data中的数据就会自动发生改变。**lazy修饰符可以让数据在失去焦点或者回车时才会更新**。

##### number修饰符

默认情况下，在输入框中无论我们输入的是字母还是数字，都会被当做字符串类型进行处理。但是如果我们希望处理的是数字类型，那么最好直接将内容当做数字处理。number修饰符可以让在输入框中输入的内容自动转成数字类型。

##### trim修饰符

如果输入的内容首尾有很多空格，通常我们希望将其去除的话，**trim修饰符可以过滤内容左右两边的空格**。

```html
<div>
  <!--2.修饰符: number-->
  <input type="number" v-model.number="age">
  <h2>{{age}}-{{typeof age}}</h2>

  <!--3.修饰符: trim-->
  <input type="text" v-model.trim="name">
  <h2>您输入的名字:{{name}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      age: 0,
      name: ''
    }
  })

  var age = 0
  age = '1111'
  age = '222'
</script>
```

### 事件监听

功能: 绑定指定事件名的回调函数 
完整写法: v-on:keyup='xxx' 
​		 v-on:keyup='xxx(参数)' 
​		 v-on:keyup.enter='xxx' 
简洁写法: @keyup='xxx' ==>@是`v-on`的语法糖，方法名的`()`在	`@事件`中可以省略
​		 @keyup.enter='xxx'

#### v-on的基本使用

```html
<h2>绑定事件监听</h2> 
		<div id="app">
			<button v-on:click="handleClick">点我</button>
            <!-- 简写 -->
			<button @click="handleClick">点我 2</button>
		</div>
		
		<script type="text/javascript"> 
		new Vue({ 
			el: '#app',  
			methods: { 
				handleClick () { 
					alert('处理点击') 
					} 
				} 
		}) 
</script>
```

#### v-on的参数传递

```html
  <div id="app">
    <!-- 事件没传参 -->
    <button @click="btnClick">按钮1</button>
    <button @click="btnClick()">按钮2</button>
    <!-- 事件调用方法传参，写函数时候省略小括号，但是函数本身需要传递一个参数 -->
    <button @click="btnClick2(123)">按钮3</button>
    <button @click="btnClick2()">按钮4</button>
    <button @click="btnClick2">按钮5</button>
    <!-- 事件调用时候需要传入event还需要传入其他参数 -->
    <button @click="btnClick3($event,123)">按钮6</button>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      methods:{
        btnClick(){
          console.log("点击XXX");
        },
        btnClick2(value){
          console.log(value+"----------");
        },
        btnClick3(event,value){
          console.log(event+"----------"+value);
        }
      }
    })
  </script>
```

按钮4调用`btnClick2(value){}`，此时为`undefined`。
按钮5调用时省略了()，会自动传入原生event事件。
按钮6调用时需要传入**某个参数**和**event事件**，可以通过`$event`传入事件

#### v-on的修饰符

`.stop`修饰的事件不会传播，不会冒泡到上层，调用了`event.stopPropagation()`。
`.prevent` 修饰的事件会调用`event.preeventDefault`**阻止默认行为**。
`.enter`修饰的事件会监听键盘的`enter`事件

```html
  <div id="app">
    <!--1. .stop的使用，btn的click事件不会传播，不会冒泡到上层，调用event.stopPropagation() -->
    <div @click="divClick">
        <button @click.stop="btnClick">按钮1</button>
    </div>
    <!-- 2. .prevent 调用event.preeventDefault阻止默认行为  -->
    <form action="www.baidu.com">
      <button type="submit" @click.prevent="submitClick">提交</button>
    </form>
    <!--3. 监听键盘的事件 -->
    <input type="text" @keydown.enter="keyup">
  </div>

  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      methods:{
        btnClick(){
          console.log("点击button");
        },
        divClick(){
          console.log("点击div");
        },
        submitClick(){
          console.log("提交被阻止了")
        },
        keyup(){
          console.log("keyup点击")
        }
      }
    })
  </script>
```

### 计算属性和侦听器

在 computed 属性对象中定义**计算属性**的方法。然后在**标签属性中**使用`方法名`或在**页面中**使用{&#123;方法名&#125;}来显示计算的结果。

性能上`computed`明显比`methods`好。而methods，即使数据没有改变，也需要再次执行。**计算属性有缓存**，只有关联属性改变才会再次计算。

#### 计算属性的基本使用

```html
		<div id="app">
			姓：<input type="text" placeholder="First Name" v-model="firstName"/><br>
			名：<input type='text' placeholder="Last Name" v-model="lastName"/><br>
            <!-- 计算属性 -->
			姓名 1(单向): <input type="text" placeholder="Full Name" v-model="fullName1"><br>
            <!-- 方法 -->
    		<h2>{{getFullName()}}</h2>
		</div>
		
		<script type="text/javascript"> 
		new Vue({ 
			el: '#app',  
			data: {
				firstName: 'A',
				lastName: 'B'
			},
			computed: {
				fullName1: function () { 
					return this.firstName + " " + this.lastName 
				}
			},
            methods: {
        		getFullName(){
          		return this.firstName + " " + this.lastName
        	}
		}) 
		</script>
```

#### 计算属性的复杂使用

```javascript
		  <div id="app">
		    <h2>总价格：{{totalPrice}}</h2>
		  </div>

		  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
		  <script>
		    const app = new Vue({
		      el:"#app",
		      data:{
		        books:[
		          {id:110,name:"JavaScript从入门到入土",price:119},
		          {id:111,name:"Java从入门到放弃",price:80},
		          {id:112,name:"编码艺术",price:99},
		          {id:113,name:"代码大全",price:150},
		        ]
		      },
		      computed: {
				  totalPrice() {
					  let total = 0;
					  for (let i = 0; i < this.books.length; i ++) {
						  total += this.books[i].price;
					  }
					  return total;
				  }
			  }
		    })
		  </script>
```

#### 计算属性的setter和getter

```html
<div id="app">
	<h1>计算属性：computed的getter/setter</h1>
	<h2>fullName</h2>
	{{fullName}}
	<h2>firstName</h2>
	{{firstName}}
	<h2>lastName</h2>
	{{lastName}}
</div>

<script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
<script>
	var app = new Vue({
		el:"#app",
		data:{
			firstName:"zhang",
		    lastName:"san",
		    },
		computed: {
			fullName: {
				get: function () {
					return this.firstName + " " + this.lastName;
				},
                <!-- set方法可以设置值，但一般不用 -->
				set: function (value) {
					var list = value.split(' ');
					this.firstName = list[0];
					this.lastName = list[1];
				}
			}
		}
    });
</script>
```

#### 计算属性之侦听器watch

`computed`范围在vue实例内，修改vue实例外部对象，不会重新计算渲染。
但是如果先修改了vue实例外对象，再修改vue的`computed`，那么vue实例外部对象的值也会重新渲染。

```html
        <div id="app">
            <h1>侦听器：watch</h1>
            {{watchFullName}}
            <h1>年龄</h1>
            {{age}}
        </div>
		
		<script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
        <script>
            var other = 'This is other';
            var app = new Vue({
                el:"#app",
                data:{
                firstName:"zhang",
                lastName:"san",
                watchFullName:"zhangsan",
                age:18,
                },
                watch: {
                    firstName:function(newFirstName, oldFirstName){
                        console.log("firstName触发了watch,newFirstName="+newFirstName+",oldFirstName="+oldFirstName)
                        this.watchFullName = this.firstName+this.lastName+","+other
                    },
                    lastName:function(newLastName, oldLastName){
                        console.log("lastName触发了watch,newLastName="+newLastName+",oldLastName="+oldLastName)
                        this.watchFullName = this.firstName+this.lastName+","+other
                    }  
                }
            });
        </script>
```

`computed`通常监听多个变量，watch监听数据变化，一般只监听一个变量或数组。
watch(`异步场景`)，computed(`数据联动`)

### 条件判断

#### v-if、v-else、v-else-if

v-if用于条件判断，判断DOM元素是否显示。
v-if的变量值为**布尔值**，为true才渲染DOM
v-if、v-else、v-else-if联合使用相当于if、elseif、else，但是在条件比较多的时候建议使用计算属性。 

```html
  <div id="app">
    <h2 v-if="isFlag">isFlag为true显示这个</h2>
    <h2 v-show="isShow">isShow为true是显示这个</h2>
    <div v-if="age<18">小于18岁未成年</div>
    <div v-else-if="age<60">大于18岁小于60岁正值壮年</div>
    <div v-else="">大于60岁,暮年</div>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        isFlag:true,
        isShow:false,
        age:66
      }
    })
  </script>
```

#### v-if的示例

点击按钮切换登录方式的示例

```html
<div id="app">
    <span v-if="isUser">
      <label for="username">用户账号</label>
      <input type="text" id="username" placeholder="请输入用户名" >
    </span>
    <span v-else="isUser">
        <label for="email">用户邮箱</label>
        <input type="text" id="email" placeholder="请输入用户邮箱" >
    </span>
    <button @click="isUser=!isUser">切换类型</button>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        isUser:true
      }
    })
  </script>
```

该示例使用`v-if`和`v-else`选择渲染指定的Dom，点击按钮对`isUser`变量取反。
存在的问题：切换输入框后，输入框未自动清空

解决方式：给对应的dom元素加上`key`值，并保证`key`不同。

```html
<input type="text" id="username" placeholder="请输入用户名" key="username">
<input type="text" id="email" placeholder="请输入用户邮箱" key="email">
```

出现问题的原因：`input`输入框被复用了

1. vue在进行DOM渲染是，处于性能考虑，会复用已经存在的元素，而不是每次都创建新的DOM元素。
2. 在上面demo中，Vue内部发现原来的input元素不再使用，所以直接将其映射对应虚拟DOM，用来复用。

#### v-show

`v-show`的变量也是布尔值，为true才显示内容，类似CSS的`display` 
`v-show`看似和`v-if`实现一样的效果，但是内部`v-show`只是用css将操作的元素**隐藏显示**，而`v-if`是新增和删除元素。

```html
  <div id="app">
    <h2 v-show="isFlag">v-show只是操作元素的style属性display，不管怎样都会被创建</h2>
    <h2 v-if="isFlag">v-if是新增和删除dom元素</h2>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        isFlag:true
      }
    })
  </script>
```

### 遍历循环

#### v-for遍历数组

```html
		<div id='app'>
			<!-- 没有使用索引的v-for -->
			<ul>
				<li v-for='item in names'>{{item}}</li>
			</ul>
			<!-- 使用了索引的v-for -->
			<ul>
				<li v-for='(item,index) in names'>{{index +':' + item}}</li>
			</ul>
		</div>
		
		<script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
		<script type='text/javascript'>
			const vm = new Vue({
				el: '#app',
				data: {
					names: ['aaa', 'bbb', 'ccc']
				}
			})
		</script>
```

一般情况下，需要使用索引值。
`<li v-for="(item,index) in names" >{&#123;index+":"+item&#125;}</li>`index表示**索引**，item表示当前遍**历的元素**。

#### v-for遍历对象

```html
			<div id='app'>
				<!-- 1.遍历过程没有使用index索引-->
				<!-- 格式为：key-value -->
				<ul>
					<li v-for='(item,key) in user'>{{key + ':' + item}}</li>
				</ul>
				<!-- 2.遍历过程使用了ndex索引-->
				<!-- 格式为：key-value-index -->
				<ul>
					<li v-for='(item,key,index) in user'>{{key + ':' + index + ':' + item}</li>
				</ul>
			</div>
			
			<script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
			<script type="text/javascript">
				const vm = new Vue({
					el: '#app',
					data: {
						user: {
							name: "张三",
							age: 18,
							identity: "法外狂徒"
						}
					}
				})
			</script>
```

`item`是当前对象的属性值，`index`是索引(下标)，`key`是当前对象的属性名

#### 组件的key属性

官方推荐我们在使用v-for时，给对应的元素或组件添加上一个`:key`属性。

- 我们需要使用key来**给每个节点做一个唯一标识**
- Diff算法就可以正确的识别此节点口找到正确的位置区插入新的节点。
- key的作用主要是为了**高效的更新虚拟DOM**

#### 检测数组更新的响应式方法

因为Vue是响应式的，所以当数据发生变化时，Vue会自动检测数据变化，使视图会发生对应的更新。Vue中包含了一组**观察数组编译的方法**，使用它们**改变数组也会触发视图的更新**。

1. btn2按钮是通过索引值修改数组的值，这种情况，数组letters变化，**DOM不会变化**。
2. 而数组的方法，例如`push()`、`pop()`、`shift()`、`unshift()`、`splice()`、`sort()`、`reverse()`等方法修改数组的数据，**DOM元素会随之修改**。

```html
  <div id="app">
    <!-- 数组的响应式方法 -->
    <ul>
      <li v-for="item in letters">{{item}}</li>
    </ul>
    <button @click="btn1">push</button><br>
    <button @click="btn2">通过索引值修改数组</button>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    const app = new Vue({
      el:"#app",
      data:{
        letters:['a','b','c','d','e']
      },
      methods: {
        btn1(){
          //1.push
          // this.letters.push('f')
          //2.pop()删除最后一个元素
          this.letters.pop()
          //3.shift()删除第一个
          //this.letters.shift()
          //4.unshift()添加在最前面,可以添加多个
          //this.letters.unshift('aaa','bbb','ccc')
          //5.splice():删除元素/插入元素/替换元素
          //splice(1,1)在索引为1的地方删除一个元素,第二个元素不传，直接删除后面所有元素
          //splice(index,0,'aaa')在索引index后面删除0个元素，加上'aaa',
          //splice(1,1,'aaa')替换索引为1的后一个元素为'aaa'
          // this.letters.splice(2,0,'aaa')
          //6.sort()排序可以传入一个函数
          //this.letters.sort()
          //7.reverse()反转
          // this.letters.reverse()

        },
        btn2(){
          this.letters[0]='f'
        }
      }
    })
  </script>
```

 