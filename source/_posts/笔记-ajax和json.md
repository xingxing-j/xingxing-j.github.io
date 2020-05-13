---
title: 笔记-ajax和json
date: 2020-04-20 19:35:06
categories: 后端学习
tags:
- [ajax]
- [json]
---

暂时就整理成这样了。ajax和json的简单学习

<!-- more -->

# ajax

## 简述

AJAX 即“Asynchronous Javascript And XML”（**异步 JavaScript 和 XML**），是指一种创建交互式网页应用的网页开发技术。ajax 是一种浏览器**通过 js 异步发起请求**，局部更新页面的技术。
Ajax 请求的局部更新，**浏览器地址栏不会发生变化**。局部更新不会舍弃原来页面的内容。

## 原生ajax请求示例

### 使用步骤

- **创建对象**：**var xmlhttprequest= new XMLHttpRequest()**

- **open连接**

  - **xmlhttprequest.open(a,b,c)**
  - a:表示**请求方式**，b:**请求地址**；c:**是否异步**，true/false，默认是true
  - 请求地址后面可拼接键值对，例：/ajax?username=xxx&password=xxx

- **send发送请求**
- 
  - **xmlhttprequest.send(null)**；参数是请求时给Servlet传递的参数，如没有,为null。
  - 可为**post**传递参数，但需要设置一下，**xmlhttprequest.setRequestHeader("content-type", "application/xxx-www-form-urlencoded");**

  - **此处待视频验证**
```javascript
xmlhttprequest.setRequestHeader("content-type", "application/xxx-www-form-urlencoded");
xmlhttprequest.send("userName=" + userName + "&password=" + password);// 不能加？
```

### 示例
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="pragma" content="no-cache" />
        <meta http-equiv="cache-control" content="no-cache" />
        <meta http-equiv="Expires" content="0" />
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Insert title here</title>
        <script type="text/javascript">
            // 在这里使用 javaScript 语言发起 Ajax 请求，访问服务器 AjaxServlet 中 javaScriptAjax
            function ajaxRequest() {
                
                // 1、我们首先要创建 XMLHttpRequest
                var xmlhttprequest = new XMLHttpRequest();
                // 2、调用 open 方法设置请求参数
                xmlhttprequest.open("GET","http://localhost:8080/16_json_ajax_i18n/ajaxServlet?action=javaScriptAjax",true)
                // 4、在 send 方法前绑定 onreadystatechange 事件，处理请求完成后的操作。
                xmlhttprequest.onreadystatechange = function(){
                    if (xmlhttprequest.readyState == 4 && xmlhttprequest.status == 200) {
                        var jsonObj = JSON.parse(xmlhttprequest.responseText);
                        // 把响应的数据显示在页面上
                        document.getElementById("div01").innerHTML = "编号：" + jsonObj.id + " , 姓名：" + jsonObj.name;
                    }
                }
                // 3、调用 send 方法发送请求
                xmlhttprequest.send();
            }
        </script>
</head>
<body>
    <button onclick="ajaxRequest()">ajax request</button>
    <div id="div01"></div>
</body>
</html>
```

#### XMLHttpRequest对象的属性

- **onreadystatechange**：回调函数
- **responseText**：响应格式为String
- **responseXML**：响应格式为XML
- **status**：响应状态，只有**200**才表示响应正常
- **readyState**：请求状态，只有**4**才代表请求完毕
  - readystate的五种状态
    - 0，表示XMLHttpRequest对象创建，open方法还没调用
    - 1，open方法调用，send方法未调用
    - 2，send方法调用，获得了响应头，未获得了响应体
    - 3，获取了响应体
    - 4，响应结束

## jQuery 中的ajax请求

### $.ajax 方法

- **url** 表示请求的地址
- **type** 表示请求的类型 GET 或 POST 请求
- **data** 表示发送给服务器的数据
  - 格式有两种：
  - 一：name=value&name=value
  - 二：{key:value}
- **success** 请求成功，**响应的回调函数**
- **dataType** 响应的数据类型
  - 常用的数据类型有：
    - text 表示纯文本
    - xml 表示 xml 数据
    - json 表示 json 对象

```javascript
$("#ajaxBtn").click(function(){
    $.ajax({
        url:"http://localhost:8080/16_json_ajax_i18n/ajaxServlet",
        // data:"action=jQueryAjax",
        data:{action:"jQueryAjax"},
        type:"GET",
        success:function (data) {
            // alert("服务器返回的数据是：" + data);
            // var jsonObj = JSON.parse(data);
            $("#msg").html("编号：" + data.id + " , 姓名：" + data.name);
        },
        dataType : "json"
    });
});
```

### $.get 方法和\$.post 方法

- **url** 请求的 url 地址
- **data** 发送的数据
- **callback** 成功的**回调函数**
- **type** 返回的数据类型

```javascript
// ajax--get 请求
$("#getBtn").click(function(){
    $.get("http://localhost:8080/16_json_ajax_i18n/ajaxServlet", "action=jQueryGet", function (data) {
        $("#msg").html(" get 编号：" + data.id + " , 姓名：" + data.name);
    }, "json");
});

// ajax--post 请求
$("#postBtn").click(function(){
    $.post("http://localhost:8080/16_json_ajax_i18n/ajaxServlet", "action=jQueryPost", function (data) {
        $("#msg").html(" post 编号：" + data.id + " , 姓名：" + data.name);
    }, "json");
});
```

# json

JSON (**JavaScript Object Notation**) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。JSON
采用完全独立于语言的文本格式，而且很多语言都提供了对 json 的支持（包括 C, C++, C#, Java, JavaScript, Perl, Python等）。 这样就使得 JSON 成为理想的数据交换格式。

### JSON 在 JavaScript 中的使用

#### json 的定义

json 是由**键值对组成**，并且由**大括号包围**。每个**键由引号引起来**，键和值之间使用冒号进行分隔，
多组键值对之间进行逗号进行分隔。

```javascript
var jsonObj = {
"key1":12,
"key2":"abc",
"key3":true,
"key4":[11,"arr",false],
    
"key5":{
"key5_1" : 551,
"key5_2" : "key5_2_value"
},
    
"key6":[{
"key6_1_1":6611,
"key6_1_2":"key6_1_2_value"},{
"key6_2_1":6621,
"key6_2_2":"key6_2_2_value"
}]
};
```

#### json 的访问

json 本身是一个对象。json 中的 key 我们可以理解为是对象中的一个属性。
json 中的 key 访问就跟访问对象的属性一样： **json 对象.key**

```javascript
alert(typeof(jsonObj));// object json 就是一个对象
alert(jsonObj.key1); //12
alert(jsonObj.key2); // abc
alert(jsonObj.key3); // true
alert(jsonObj.key4);// 得到数组[11,"arr",false]

// json 中 数组值的遍历
for(var i = 0; i < jsonObj.key4.length; i++) {
alert(jsonObj.key4[i]);
}
alert(jsonObj.key5.key5_1);//551
alert(jsonObj.key5.key5_2);//key5_2_value
alert( jsonObj.key6 );// 得到 json 数组
// json数组中取出来的每一个元素都是 json 对象
var jsonItem = jsonObj.key6[0];
// alert( jsonItem.key6_1_1 ); //6611
alert( jsonItem.key6_1_2 ); //key6_1_2_value
```

#### json的两个常用方法

- **JSON.stringify()** 把 json 对象转换成为 json 字符串
- **JSON.parse()** 把 json 字符串转换成为 json 对象

json 的存在有两种形式。对象的形式存在，我们叫它 **json 对象**；字符串的形式存在，我们叫它 **json 字符串**。

- 一般我们要操作 json 中的数据的时候，需要 json 对象的格式。
- 一般我们要在客户端和服务器之间进行数据交换的时候，使用 json 字符串。

```javascript
// 把 json 对象转换成为 json 字符串
var jsonObjString = JSON.stringify(jsonObj); // 特别像 Java 中对象的 toString
alert(jsonObjString)

// 把 json 字符串。转换成为 json 对象
var jsonObj2 = JSON.parse(jsonObjString);
alert(jsonObj2.key1);// 12
alert(jsonObj2.key2);// abc
```

### JSON 在 java 中的使用

#### javaBean 和 json 的互转

```java
@Test
public void test1(){
    Person person = new Person(1,"国哥好帅!");
    // 创建 Gson 对象实例
    Gson gson = new Gson();
    // toJson 方法可以把 java 对象转换成为 json 字符串
    String personJsonString = gson.toJson(person);
    System.out.println(personJsonString);
    // fromJson 把 json 字符串转换回 Java 对象
    // 第一个参数是 json 字符串
    // 第二个参数是转换回去的 Java 对象类型
    Person person1 = gson.fromJson(personJsonString, Person.class);
    System.out.println(person1);
}
```

#### List 和 json 的互转

```java
@Test
public void test2() {
    List<Person> personList = new ArrayList<>();
    personList.add(new Person(1, "国哥"));
    personList.add(new Person(2, "师傅"));
    Gson gson = new Gson();
    // 把 List 转换为 json 字符串
    String personListJsonString = gson.toJson(personList);
    System.out.println(personListJsonString);
    List<Person> list = gson.fromJson(personListJsonString, new PersonListType().getType());
    System.out.println(list);
    Person person = list.get(0);
    System.out.println(person);
}
```

#### map 和 json 的互转

```java
@Test
public void test3(){
    Map<Integer,Person> personMap = new HashMap<>();
    personMap.put(1, new Person(1, "国哥好帅"));
    personMap.put(2, new Person(2, "康师傅也好帅"));
    Gson gson = new Gson();
    // 把 map 集合转换成为 json 字符串
    String personMapJsonString = gson.toJson(personMap);
    System.out.println(personMapJsonString);
    // Map<Integer,Person> personMap2 = gson.fromJson(personMapJsonString, new PersonMapType().getType());
    Map<Integer,Person> personMap2 = gson.fromJson(personMapJsonString, new
    TypeToken<HashMap<Integer,Person>>(){}.getType());
    System.out.println(personMap2);
    Person p = personMap2.get(1);
    System.out.println(p);
}
```



