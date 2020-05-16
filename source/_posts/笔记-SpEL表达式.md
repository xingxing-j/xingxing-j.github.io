---
title: 笔记-SpEL表达式
date: 2020-05-14 15:01:11
categories: 后端学习
tags:
- [Spring]
- [表达式]
---

SpEL表达式的使用说明

<!-- more -->

# SpEL

## 简述

**SpringExpression Language**，Spring表达式语言，简称SpEL。支持运行时查询并可以操作对象图。
和JSP页面上的**EL**表达式、Struts2中用到的**OGNL**表达式一样，SpEL根据JavaBean风格的**getXxx()**、**setXxx()**方法定义的属性**访问对象图，**完全符合我们熟悉的操作习惯。

## 语法

SpEL使用`#{…}`作为定界符，所有在大框号中的字符都将被认为是SpEL表达式。

### 使用

#### 使用字面值

- 整数
```xml
<property name="count" value="#{5}"/>
```
- 小数
```xml
- <property name="frequency" value="#{89.7}"/>
```
- 科学计数法
```xml
- <property name="capacity" value="#{1e4}"/>
```
- String类型
```xml
<property name=“name” value="#{'Chuck'}"/>
<property name='name'  value='#{"Chuck"}'/>
```
- Boolean
```xml
<property name="enabled" value="#{false}"/>
```
#### 引用其他bean

此处待验证

```xml
<bean id="emp04" class="com.xxx.parent.bean.Employee">
	<property name="empId" value="1003"/>
	<property name="empName" value="Kate"/>
	<property name="age" value="21"/>
	<property name="detp" value="#{dept}"/>
</bean>
```

#### 引用其他bean的属性值作为自己某个属性的值

```xml
<bean id="emp05" class="com.xxx.parent.bean.Employee">
	<property name="empId" value="1003"/>
	<property name="empName" value="Kate"/>
	<property name="age" value="21"/>
	<property name="deptName" value="#{dept.deptName}"/>
</bean>
```

#### 调用非静态方法

```xml
<bean id="employee" class="com.atguigu.spel.bean.Employee">
	<!-- 通过对象方法的返回值为属性赋值 -->
	<property name="salayOfYear" value="#{salaryGenerator.getSalaryOfYear(5000)}"/>
</bean>
```

#### 调用静态方法

格式：**#{T(全类名).静态方法名(参数)}**

```xml
<bean id="employee" class="com.atguigu.spel.bean.Employee">
	<!-- 在SpEL表达式中调用类的静态方法 -->
	<property name="circle" value="#{T(java.lang.Math).PI*20}"/>
</bean>
```

#### 支持的运算符

- 算术运算符：**+、-、*、/、%、^**
- 字符串连接：**+**
- 比较运算符：**<、>、==、<=、>=、lt、gt、eq、le、ge**
- 逻辑运算符：**and, or, not, |**
- 三目运算符：**判断条件?判断结果为true时的取值:判断结果为false时的取值**
- 正则表达式：**matches**

