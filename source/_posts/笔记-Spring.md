---
title: 笔记-Spring
date: 2020-04-20 19:46:11
categories: 后端学习
tags:
- [Spring]
- [框架]
---

施工中

<!-- more -->

# Spring

## 概述

Spring是一个开源框架。简化了企业级开发。Spring是一个**IOC**(**DI**)和**AOP**容器框架。

## Spring特点

- **非侵入式**：基于Spring开发的应用中的对象可以不依赖于Spring的API
- **依赖注入**：DI——**Dependency Injection**，反转控制(IOC)最经典的实现。
- **面向切面编程**：AspectOriented Programming——AOP
- **容器**：**Spring是一个容器**，因为它包含并且管理应用对象的生命周期
- **组件化**：Spring实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用XML和Java注解组合这些对象。
- **一站式**：在IOC和AOP的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上Spring自身也提供了表述层的SpringMVC和持久层的SpringJDBC）

## HelloWorld示例

### 导包
- **spring-beans.jar**：处理Bean的jar \<Bean>
- **spring-context.jar**：处理spring上下文的jar  \<context>
- **spring-core.jar**：spring核心jar（必需）
- **spring-expression.jar**：spring表达式 
- **spring-aop.jar**：开发AOP特性时需要的JAR
- **commons-logging.jar**：第三方提供的日志jar包

### 创建实体类

```java
public class User {
    private Integer uId;
    private String uName;
    private Integer uAge;
    private String desc;
	// getter、setter和toString方法略
}
```

### 编写Spring配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       default-autowire="byName"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

    <!-- 使用bean元素定义一个由IOC容器创建的对象 -->
    <!-- class属性指定用于创建bean的全类名 -->
    <!-- id属性指定用于引用bean实例的标识 -->
    <bean id="user" class="com.xxx.cn.entity.User">
        <!-- 使用property子元素为bean的属性赋值 -->
        <property name="uId" value="002"/>
        <property name="uName" value="王二"/>
        <property name="uAge" value="20"/>
        <property name="desc" value="是个傻子"/>
    </bean>
</beans>
```

之后要在IDEA的**Project Structure**的**Modules**里配置一下Spring应用上下文，如下图

![IDEA中添加Spring应用上下文](笔记-Spring/IDEA中添加Spring应用上下文.png)

### 编写测试类测试

注：**Spring在创建IOC容器对象时，就已经完成了bean的创建和属性的赋值**。

```java
public class DemoTest {

    public static void main(String[] args) {
        //1.创建IOC容器对象
        ApplicationContext iocContainer =
                new ClassPathXmlApplicationContext("applicationcontext.xml");
        //2.根据ID值获取bean实例对象
        User user = (User) iocContainer.getBean("user");
        //3.打印bean
        System.out.println(user);
    }
}
```

## IOC和DI          

### IOC(Inversion of Control)

反转控制。Spring反转了资源的获取方向，改**由容器主动的将资源推送给需要的组件**，开发人员不需要知道容器是如何创建资源对象的，只需要提供接收资源的方式即可。

### DI(Dependency Injection)

依赖注入。IOC的另一种表述方式：即组件以一些预先定义好的方式(例如：setter 方法)接受来自于容器的资源注入。将属性值 注入给了属性，将属性注入给了bean，将bean注入给了ioc容器。

### IOC容器在Spring中的实现

在通过IOC容器读取Bean的实例之前，需要先将IOC容器本身实例化。**Spring提供了IOC容器的两种实现方式**

- **BeanFactory**：IOC容器的基本实现，是Spring内部的基础设施，是**面向Spring本身**的，不是提供给开发人员使用的
- **ApplicationContext**：BeanFactory的子接口，提供了更多高级特性。面向Spring的使用者，几乎所有场合都使用ApplicationContext，而不是底层的BeanFactory
  - 在**初始化时就创建单例的bean**，也可以通过配置的方式指定创建的Bean是多实例的
  - **ClassPathXmlApplicationContext**，对应**类路径下**的XML格式的配置文件
  - **FileSystemXmlApplicationContext**，对应**文件系统中**的XML格式的配置文件
- **ConfigurableApplicationContext**，是ApplicationContext的子接口，包含一些扩展方法。**refresh()**和**close()**让ApplicationContext具有启动、关闭和刷新上下文的能力。
- **WebApplicationContext**，专门为WEB应用而准备的，它允许从相对于WEB根目录的路径中完成初始化工作

## 获取Bean的两种方式

- 通过Bean的**ID值**从IOC容器中获取

```java
User user = (User) iocContainer.getBean("user");
```

- 通过Bean的类型从IOC容器中获取，但如果同一个类型的bean在XML文件中配置了多个，则获取时会抛出异常，所以同一个类型的bean在容器中必须是唯一的。

```java
User user = (User) iocContainer.getBean(User.class);
```

## Bean的赋值方式

### 通过Bean的**setXxx()**方法赋值

下面的property标签除了name、value属性外，还可以用ref属性引用外部的Bean。

```xml
<bean id="user" class="com.xxx.cn.entity.User">
    <!-- 使用property子元素为bean的属性赋值 -->
    <property name="uId" value="002"/>
    <property name="uName" value="王二"/>
    <property name="uAge" value="20"/>
    <property name="desc" value="是个傻子"/>
</bean>
```

![setXxx()赋值的方式](笔记-Spring/通过setXxx()方式赋值.png)

### 通过Bean的构造器赋值

#### 基本写法

这种基本的写法，在bean标签里使用了**constructor-arg**标签，其属性有name，有value

```xml
<bean id="book" class="com.xxx.spring.bean.Book">
    <constructor-arg name="id" value= "10010"/>
    <constructor-arg name="bookName" value= "Book01"/>
    <constructor-arg name="authorName" value= "Author01"/>
    <constructor-arg name="number" value= "20.2"/>
</bean >
```

#### 省略name属性的写法

**这种写法需严格按照参数顺序**

```xml
<bean id="book" class="com.xxx.spring.bean.Book">
    <constructor-arg value= "10010"/>
    <constructor-arg value= "Book01"/>
    <constructor-arg value= "Author01"/>
    <constructor-arg value= "20.2"/>
</bean >
```

#### 通过索引值指定参数位置

这种好像也可以省略name属性，**待验证**

```xml
<bean id="book" class="com.xxx.spring.bean.Book">
    <constructor-arg value= "10010" index ="0"/>
    <constructor-arg value= "Book01" index ="1"/>
    <constructor-arg value= "Author01" index ="2"/>
    <constructor-arg value= "20.2" index ="3"/>
</bean >
```

#### 通过类型不同区分重载的构造器

这种好像也可以省略name属性，**待验证**

```xml
<bean id="book" class="com.xxx.spring.bean.Book">
    <constructor-arg value= "10010" index ="0" type="java.lang.Integer" />
    <constructor-arg value= "Book01" index ="1" type="java.lang.String" />
    <constructor-arg value= "Author01" index ="2" type="java.lang.String" />
    <constructor-arg value= "20.2" index ="3" type="java.lang.Double" />
</bean >
```

### Bean的级联属性赋值

```xml
<bean id="action" class="com.xxx.spring.ref.Action">
    <property name="service" ref="service"/>
    <!-- 设置级联属性(了解) -->
    <property name="service.dao.dataSource" value="DBCP"/>
</bean>
```

### p名称空间赋值

为了简化XML文件的配置，越来越多的XML文件采用属性而非子元素配置信息。Spring从2.5版本开始引入了一个新的**p命名空间**，可以通过\<bean>元素属性的方式配置Bean的属性。使用p命名空间后，基于XML的配置方式将进一步简化。

```xml
<bean 
	id="studentSuper" 
	class="com.xxx.helloworld.bean.Student"
	p:studentId="2002" p:stuName="Jerry2016" p:age="18" />
```

## bean标签内可以用的值

### 字面值

- 可以使用字符串表示的值，可以通过value属性或value子节点的方式指定
- 基本数据类型及其封装类、String等类型都可以采取字面值注入的方式
- 若字面值中包含特殊字符，可以使用<![CDATA[]]>把字面值包裹起来

### 赋Null值

```xml
<bean class="com.xxx.spring.bean.Book" id="bookNull" >
    <property name= "bookId" value ="2000"/>
    <property name= "bookName">
        <null/>
    </property>
    <property name= "author" value ="nullAuthor"/>
    <property name= "price" value ="50"/>
</bean >
```

### 引用外部声明的bean

```xml
<bean id="shop" class="com.xxx.spring.bean.Shop" >
    <property name= "book" ref ="book"/>
</bean >
```

### 使用内部bean

当bean实例**仅仅给一个特定的属性使用**时，可以将其声明为内部bean。
内部bean声明直接包含在\<property>或\<constructor-arg>元素里，不需要设置任何id或name属性。
**内部bean不能使用在任何其他地方**。

```xml
<bean id="shop" class="com.xxx.spring.bean.Shop" >
    <property name= "book">
        <bean class= "com.xxx.spring.bean.Book" >
           <property name= "bookId" value ="1000"/>
           <property name= "bookName" value="innerBook"/>
           <property name= "author" value="innerAuthor" />
           <property name= "price" value ="50"/>
        </bean>
    </property>
</bean >
```

## 集合的赋值

### List

Set，数组的赋值方式跟Llist差不多。

#### 通过\<value>指定简单的常量值

```xml
<bean id="shop" class="com.xxx.spring.bean.Shop" >
    <property name= "categoryList">
        <!-- 以字面量为值的List集合 -->
        <list>
            <value>历史</value >
            <value>军事</value >
        </list>
    </property>
</bean >
```

#### 通过\<ref>指定对其他Bean的引用

```xml
<bean id="shop" class="com.xxx.spring.bean.Shop" >
    <property name= "bookList">
        <!-- 以bean的引用为值的List集合 -->
        <list>
            <ref bean= "book01"/>
            <ref bean= "book02"/>
        </list>
    </property>
</bean >
```

#### 通过\<bean>指定内置bean定义

```xml
<bean id="shop" class="com.atguigu.spring.bean.Shop" >
    <property name= "bookList">
        <!-- 内置bean的List集合 -->
        <list>
            <bean class="com.atguigu.spring.bean.Book" p:bookName="西游记"/>
        </list>
    </property>
</bean >
```

#### 通过\<null/>指定空元素。甚至可以内嵌其他集合

这个就不举例了，自己想象吧

###Map

\<map>标签里可以使用多个\<entry>作为子标签。一个\<entry>标签代表一个键值对 。

#### 套娃写法一

```xml
<bean id="cup" class="com.xxx.spring.bean.Cup">
    <property name="maps">
        <map>
            <entry key="name" value="张三"/>
            <entry key="age" value="17"/>
            <entry key="key03" value-ref="张三"/>
            <entry key="key04">
                <bean class="com.xxx.bean.Car">
                    <property name="carName" value="xxx"/>
                </bean>
            </entry>
        </map>  
    </property>
</bean>
```

#### 套娃写法二

```xml
<bean id="cup" class="com.xxx.spring.bean.Cup">
    <property name="bookMap">
        <map>
            <entry>
                <key>
                    <value>bookKey01</value>
                </key>
                <ref bean="book01"/>
            </entry>
            <entry>
                <key>
                    <value>bookKey02</value>
                </key>
                <ref bean="book02"/>
            </entry>
        </map>
    </property>
</bean>
```

### Properties

使用\<props>定义java.util.Properties，该标签使用多个\<prop>作为子标签。每个\<prop>标签必须定义key属性

```xml
<bean class="com.xxx.spring.bean.DataSource" id="dataSource">
	<property name="properties">
		<props>
			<prop key="userName">root</prop>
			<prop key="password">root</prop>
			<prop key="url">jdbc:mysql:///test</prop>
			<prop key="driverClass">com.mysql.jdbc.Driver</prop>
		</props>
	</property>
</bean>
```

### 集合属性的Bean

如果集合对象配置在某个bean内部，则这个集合的配置将不能重用。要想重用需要将集合bean的配置拿到外面，供其他bean引用。**配置集合类型的bean需要引入util名称空间**

```xml
<util:list id="bookList">
	<ref bean="book01"/>
	<ref bean="book02"/>
	<ref bean="book03"/>
	<ref bean="book04"/>
	<ref bean="book05"/>
</util:list>

<util:list id="categoryList">
	<value>编程</value>
	<value>极客</value>
	<value>相声</value>
	<value>评书</value>
</util:list>
```

## 工厂创建Bean





## Bean的高级配置

### 配置信息的继承

Spring允许继承bean的配置，被继承的bean称为父bean，继承这个父bean的bean称为子bean。
子bean从父bean中继承配置，包括bean的属性配置。子bean也可以覆盖从父bean继承过来的配置。

```xml
<bean id="dept" class="com.xxx.parent.bean.Department">
	<property name="deptId" value="100"/>
	<property name="deptName" value="IT"/>
</bean>

<bean id="emp01" class="com.xxx.parent.bean.Employee">
	<property name="empId" value="1001"/>
	<property name="empName" value="Tom"/>
	<property name="age" value="20"/>

	<!-- 会被继承的属性值 -->	
	<property name="detp" ref="dept"/>
</bean>
<!-- 以emp01作为父bean，继承后可以省略公共属性值的配置 -->
<bean id="emp02" parent="emp01">
	<property name="empId" value="1002"/>
	<property name="empName" value="Jerry"/>
	<property name="age" value="25"/>
</bean>
```

#### 需要注意的点

- 父bean可以作为配置模板，也可以作为bean实例。若只想把父bean作为模板，可以设置\<bean>**abstract**属性为true，这样Spring将不会实例化这个bean。
- 如果一个bean的**class属性没有指定，则必须是抽象bean**
- 并不是\<bean>元素里的所有属性都会被继承。比如：autowire，abstract等。
- 也可以忽略父bean的class属性，让子bean指定自己的类，而共享相同的属性配置。但此时abstract必须设为true。

### bean之间的依赖         

有的时候创建一个bean的时候需要保证另外一个bean也被创建，这时我们称前面的bean对后面的bean有依赖。例如：要求创建Employee对象的时候必须创建Department。这里需要注意的是**依赖关系不等于引用关系**，Employee即使依赖Department也可以不引用它。

```xml
<bean id="emp03" class="com.xxx.parent.bean.Employee" depends-on="dept">
	<property name="empId" value="1003"/>
	<property name="empName" value="Kate"/>
	<property name="age" value="21"/>
</bean>
```

### bean的作用域         

Spring中，可以在\<bean>元素的**scope属性里设置bean的作用域**，以决定这个bean是单实例的还是多实例的。

默认情况下，Spring只为每个在IOC容器里声明的bean创建唯一一个实例，整个IOC容器范围内都能共享该实例。所有后续的getBean()调用和bean引用都将返回这个唯一的bean实例。该作用域被称为**singleton**，它是所有bean的默认作用域。

![Bean的作用域](笔记-Spring/Bean的作用域.png)

### bean的生命周期         

