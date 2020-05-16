---
title: 笔记-XML
date: 2020-05-15 11:00:06
categories: 前端学习
tags:
- [XML]
---

XML的简单了解

<!-- more -->

# XML

## 概述 

Extensible Markup Language，**可扩展**标记语言。

## 功能

存储数据

## 语法特点

- xml文档的后缀名为   .xml
- xml**第一行**必须定义为文档声明
- xml文档中**有且仅有一个根标签**
- 标签的属性值必须使用引号(单双都行)引起来
- 标签必须正确关闭
- **xml标签名称区分大小写**

## 组成部分

### 1. 文档声明

```xml
<?xml 属性列表 ?>
```

- **version**: 版本号，必须的属性
- **encoding**：编码方式。告知解析引擎当前文档使用的编码集，默认值：ISO-8859-1
- **standalone**：是否独立，取值为'yes'，表示不依赖其他文件，取值为'no'，表示依赖其他文件。不常用 

### 2、指令

仅做了解

```xml
<?xml-stylesheet type="text/css" href=" xxx.css"?>
```

### 3、标签

标签名称自定义

##### 命名规则

- 名称可包含字母数字及其他字符
- 名称**不能以数字或标点符号开头**
- 名称**不能以字母xml开头**
- 名称**不能包含空格**

### 4、属性

id属性值唯一

### 5、文本

CDATA区

区中文本会被原样展示
<![ CDATA [ 数据 ] ]>

## 约束

约束，规定了xml文档的书写规则。

#### 分类

##### DTD

简单的约束技术。

###### 内部DTD

内部DTD将约束规则定义在xml文档中。

书写格式：

```xml
<!DOCTYPE 根标签名 [

<!ELEMENT students (student+)>
<!ELEMENT student (name,age,sex)>
<!ELEMENT name (#PCDATA)>
<!ELEMENT age (#PCDATA)>
<!ELEMENT sex (#PCDATA)>
<!ATTLIST student number ID #REQUIRED>

]>
```

- 加号表示出现1次或多次
- 星号表示出现0次或多次
- \#PCDATA表示字符串
- number属性为ID属性，唯一
- \#REQUIRED表示必须出现
- ELEMENT定义标签
- ATTLIST定义属性

###### 外部DTD

外部DTD将约束规则定义在外部DTD文件中

```xml
<!-- 引入本地本地的DTD约束文件 -->
<!DOCTYPE 根标签名 SYSTEM “xxx.dtd”>
<!-- 引入本地网络上的DTD约束文件 -->
<!DOCTYPE 根标签名 PUBLIC "xxx.dtd" "dtd文件位置的URL">
```

##### Schema

复杂的约束技术

###### 引入顺序

1. 填写xml文档的根元素
2. 引入xsi前缀，xmlns:xsi="xxxxx"
3. 引入xsd文件命名空间，xsi:schemaLocation="xxxxx"
4. 为每一个xsd约束声明一个前缀，作为标识 xmlns:a="xxxxxx"(a为前缀)

## 解析

### 解析方式

#### DOM

- 将标记语言文档一次性加载进内存中，在内存中形成一颗DOM树
- 优点：操作方便，可以对文档进行CRUD的所有操作
- 缺点：很占内存

#### SAX

- 逐行读取，基于事件驱动的
- 优点：基本不占内存
- 缺点：只能读取，不能增删改

### 常见解析器

- **JAXP**：sun公司提供的解析器，支持DOM和SAX解析
- **DOM4J**：常用在服务器端
- **PULL**：Android操作系统内置解析器，支持SAX解析
- **Jsoup**：一款Java的Html解析器，可直接解析某个URL地址、HTML文本内容。可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。



