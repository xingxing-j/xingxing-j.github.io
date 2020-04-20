---
title: thymeleaf常用语法
date: 2020-04-24 01:14:43
categories: 前端学习
tags:
- [thymeleaf]
- [模板引擎]
- [语法]
---

了解下thymeleaf的常用语法

<!-- more -->

# thymeleaf的常用语法

该文参照<https://www.cnblogs.com/msi-chen/p/10974009.html>

## 0. 变量

### 0.0. 变量示例

#### 0.0.0. 编写实体类 

```java
public class User {
    String name;
    int age;
    User friend;
}
```

#### 0.0.1. 在模型中添加数据 

```java
@GetMapping("test2")
public String test2(Model model){
    User user = new User();
    user.setAge(21);
    user.setName("Jackson");
    user.setFriend(new User("李小龙", 30));
    
    model.addAttribute("user", user);
    return "hello2";
}
```

#### 0.0.2. 前端页面的编写

```html
<h1>
    你好,<span th:text="${user.name}">请跟我来</span>
</h1>
```

​	thymeleaf通过`${}`来获取model中的变量，该形式的表达式写在`th:text`的标签属性中。

### 0.1. 变量之动静结合

​	thymeleaf中所有的表达式都需要写在**"指令**"中，指令是HTML5中的自定义属性。在thymeleaf中所有指令都是以`th:`开头。在静态环境下，表达式的内容会被当做是普通字符串，浏览器会自动忽略这些指令，这样就不会报错了。

```html
<h1>
    你好,<span th:text="${user.name}">请跟我来</span>
</h1>
```

​	例如上面写的示例，不经过SpringMVC，直接放在静态环境中，会显示缺省值：“请跟我来”。

​	假如浏览器不支持Html5，可以把`th:text`换成`data-th-text`。

### 0.2. 自定义变量

#### 示例

```html
<h2>
    <p>Name: <span th:text="${user.name}">Jack</span>.</p>
    <p>Age: <span th:text="${user.age}">21</span>.</p>
    <p>friend: <span th:text="${user.friend.name}">Rose</span>.</p>
</h2>
```

​	当需要获取的数据量比较多的时候，会很麻烦。而thymeleaf提供了**自定义的变量**来解决该问题。

```html
<h2 th:object="${user}">
    <p>Name: <span th:text="*{name}">Jack</span>.</p>
    <p>Age: <span th:text="*{age}">21</span>.</p>
    <p>friend: <span th:text="*{friend.name}">Rose</span>.</p>
</h2>
```

​	上述例子，首先在h2标签上使用了`th:object="${user}"`获取user的值，并且保存。然后，在h2标签内部的任意元素上，通过`*{属性名}`的方式，获取user中的属性，这样便省去了大量的user前缀。

## 1. 方法调用

​	thymeleaf中提供了一些内置对象，并且在这些对象中提供了一些方法，来方便调用。而获取这些对象，需要使用`#对象名`来引用。

	### 一些环境相关对象

| 对象              | 作用                                          |
| ----------------- | --------------------------------------------- |
| `#ctx`            | 获取Thymeleaf自己的Context对象                |
| `#requset`        | 如果是web程序，可以获取HttpServletRequest对象 |
| `#response`       | 如果是web程序，可以获取HttpServletReponse对象 |
| `#session`        | 如果是web程序，可以获取HttpSession对象        |
| `#servletContext` | 如果是web程序，可以获取HttpServletContext对象 |

### thymeleaf提供的全局对象

| 对象         | 作用                             |
| ------------ | -------------------------------- |
| `#dates`     | 处理java.util.date的工具对象     |
| `#calendars` | 处理java.util.calendar的工具对象 |
| `#numbers`   | 用来对数字格式化的方法           |
| `#strings`   | 用来处理字符串的方法             |
| `#bools`     | 用来判断布尔值的方法             |
| `#arrays`    | 用来护理数组的方法               |
| `#lists`     | 用来处理List集合的方法           |
| `#sets`      | 用来处理set集合的方法            |
| `#maps`      | 用来处理map集合的方法            |

### 方法调用的示例

先在model中添加日期类对象

```java
@GetMapping("test3")
public String show3(Model model){
    model.addAttribute("today", new Date());
    return "hello3";
}
```

前端页面代码：

```html
<p>
    今天是：<span th:text="${#dates.format(today,'yyyy-MM-dd')}">xxxx</span>
</p>
```

## 2. 字面值

​	如果我们在**指令**中填写的字面值如：字符串、数值、布尔等，并不希望被Thymeleaf解析为变量，而是原来写什么就出现什么，该怎么办呢？

### 2.0. 字符串字面值

使用`''`单引号引用起来的内容，thymeleaf并不会认为是变量，而是一个字符串。例如：

```html
<p>
    正在观看<span th:text="'thymeleaf'">template</span>的字符串常量值
</p>
```

### 2.1. 数字字面值

数字不需要任何特殊语法，写什么就是什么，并且可以直接进行算数运算。

```html
<p>今年是<span th:text="2020">1222</span>年</p>
<p>两年后是<span th:text="2020+2">1224</span></p>
```

### 2.2. 布尔字面值

布尔类型的字面值是true或false

```html
<div th:if="true">
    你填的是true
</div>
```

## 3. 拼接

普通字符串与表达式拼接的情况如下：

```html
<span th:text="'欢迎你:' + ${user.name} + '~'"></span>
```

​	可以看出，拼接起来还是比较麻烦的，thymeleaf对此进行了优化，使用`||`即可。示例如下：

```html
<span th:text="|欢迎你:${user.name}|"></span>
```

## 4. 运算

​	`${}`内部是通过OGNL表达式引擎解析的，外部的才是通过thymeleaf的引擎解析的，因此**运算符尽量放在`${}`外进行。

### 4.0. 算术运算

支持的算术运算符有：+、-、*、/、%

```html
<span th:text="${user.age}"></span>
<span th:text="${user.age} % 2 == 0"></span>
```

### 4.1. 比较运算

支持的比较运算符有：`>`、`<`、`>=`、`<=`、`==`和`!=`。

`==`和`!=`不仅可以比较数值，还有类似于equals的功能。

还有`>`和`<`不能直接使用，因为xml解析标签，需要使用别名。

可以使用的别名有：`gt`(>)，`lt`(<)，`ge`(>=)，`le`(<=)，`not`(!)，`Also eq`(==)，`neq/ne`(!=)

### 4.2. 三目运算

```html
<span th:text="${user.sex} ? '男' : '女'"></span>
```

#### 默认值

如果取的值为空，需要做非空判断，这时候可以用表达式`?:`。

当前面的表达式的值为null时，就会使用后面的默认值。

```html
<span th:text="${user.name} ?: '二狗'"></span>
```

`?:`之间没有空格

## 5. 循环

使用`th:each`指令完成，用来遍历集合。

```html
<tr th:each="user : ${users}">
	<td th:text="${user.name}">Online</td>
    <td th:text="${user.age}">111</td>
</tr> 
```

- ${users}是要遍历的集合，可以是以下的类型：
  - Iterable，实现了Iterable接口的类
  - Enumeration，枚举
  - Interator，迭代器
  - Map，遍历得到的是Map.Entry
  - Array，数组及其他一切符合数组结果的对象

迭代的同时，还可以获取**迭代的状态对象**

```html
<tr th:each="user,stat : ${users}">
	<td th:text="${user.name}">Online</td>
    <td th:text="${user.age}">111</td>
</tr>
```

- stat对象包含以下属性：
  - index，从0开始的角标/下标/索引
  - count，元素的个数，从1开始
  - size，总元素的个数
  - current，当前遍历到的元素
  - even/odd，返回是否为奇偶，返回的是布尔值
  - first/last，返回是否为开始或最后，返回的是布尔值

## 6. 逻辑判断

thymeleaf中使用`th:if`和`th:unless`替代`if`和`else`

```html
<span th:if="${user.age} > 22">年轻</span> 
```

如果表达式的值为true，则标签则会渲染到页面，**否则不进行渲染**。

下列属性在`th:if`指令中会被认定为true：

- 表达式值为true
- 表达式值为非0数值
- 表达式值为非0字符
- 表达式值为字符串，但不是”false“，”no“，”off“
- 表达式不是布尔」字符串」数字、字符中的任何一种

其他情况包括null都会被认定为false

## 7. 分支控制switch

使用到`th:switch`和`th:case`指令

```html
<div th:switch="${user.role}">
  <p th:case="'admin'">用户是管理员</p>
  <p th:case="'manager'">用户是经理</p>
  <p th:case="*">用户是别的玩意</p>
</div>
```

一旦有一个`th:case`成立，其它的则不再判断。

`th:case="*"`表示默认，即java中switch语句中的default。

## 8. JS模板

​	模板引擎不仅可以渲染html，也可以对JS中的进行预处理。而且为了在纯静态环境下可以运行，其thymeleaf代码可以被注释起来：

```
<script th:inline="javascript">
    const user = /*[[${user}]]*/ {};
    const age = /*[[${user.age}]]*/ 20;
    console.log(user);
    console.log(age)
</script>
```

- 在script标签中通过`th:inline="javascript"`来声明这是要特殊处理的js脚本

- 语法结构：

  ```
  const user = /*[[Thymeleaf表达式]]*/ "静态环境下的默认值";
  ```

  因为Thymeleaf被注释起来，因此即便是静态环境下， js代码也不会报错，而是采用表达式后面跟着的默认值。且User对象会被直接处理为json格式。

