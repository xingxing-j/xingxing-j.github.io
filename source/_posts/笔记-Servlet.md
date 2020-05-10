---
title: 笔记-Servlet
date: 2020-04-20 19:32:06
categories: 后端学习
tags:
- [Servlet]
- [Java笔记]
---

Servlet的简单学习

<!-- more -->

# Servlet

## 0. 简述

Servlet是两单词，Server、applet的合并，即**服务器上的Java小程序**。它是JavaEE的规范之一，JavaWeb三大组件之一。
简单理解，就是服务器端的Java小程序不能随意编写，必须要实现Servlet接口(其实我也不是很懂(●—●)),然后就可以接收客户端发送过来的请求，并响应数据给客户端了。

## 1. Servlet程序的Web.xml文件配置示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
version="4.0">
<!-- servlet 标签给 Tomcat 配置 Servlet 程序 -->
<servlet>
<!--servlet-name 标签 Servlet 程序起一个别名（一般是类名） -->
<servlet-name>HelloServlet</servlet-name>
<!--servlet-class 是 Servlet 程序的全类名-->
<servlet-class>com.xxx.servlet.HelloServlet</servlet-class>
</servlet>
<!--servlet-mapping 标签给 servlet 程序配置访问地址-->
<servlet-mapping>
<!--servlet-name 标签的作用是告诉服务器，我当前配置的地址给哪个 Servlet 程序使用-->
<servlet-name>HelloServlet</servlet-name>
<!--url-pattern 标签配置访问地址 <br/>
/ 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径 <br/>
/hello 表示地址为：http://ip:port/工程路径/hello <br/>
-->
<url-pattern>/hello</url-pattern>
</servlet-mapping>
</web-app>
```

## 2. Servlet的生命周期

Servlet对象从最初的创建，方法的调用，到最后的销毁，都是由**Web容器管理**的。
默认情况下，Servlet对象在Web容器启动时不会实例化，可通过设置来改变其初始化的顺序(在web.xml文件中的servlet标签中配置load-on-startup标签)。

### 2.0. 简述Servlet生命周期

- 用户在浏览器上输入URL
- Web容器截取了请求路径
- Web容器在容器上下文中寻找请求路径对应的Servlet对象
- 如果没有找到对应的Servlet对象
  - 通过web.xml文件中配置的相关信息，得到请求路径对应的Servlet的完整类名
  - 通过反射机制调用对应Servlet的无参构造方法将此Servlet实例化
  - Web容器调用该Servlet的**init方法**进行初始化操作
  - Web容器再调用该Servlet的**service方法**来提供服务
- 若找到对应的Servlet对象，Web容器会直接调用该Servlet的**service方法**提供服务
- Web容器关闭时或webapp重新部署时或该Servlet对象长时间没有被调用，Web容器都会把该Servlet对象销毁。**在销毁之前**，Web容器会调用该Servlet对象的**destroy方法**，完成销毁前的准备

### 2.1. Servlet各种方法的运执行次数

- 构造方法只执行一次
- init方法只执行一次，执行时，Servket对象已经创建好
- service方法，用户请求一次，执行一次
- destroy方法只执行一次

## 3. ServletConfig

### 3.0. 简述

ServletConfig是一个接口，封装了对应Servlet对象的配置信息，Web容器中有它的实现类。它和Servlet对象一一对应，且在Servlet对象创建时由Web容器自动创建。

### 3.1. ServletConfig的作用

- 获取 Servlet 程序的别名 servlet-name 的值
- 获取初始化参数 init-param
- 获取 ServletContext 对象

### 3.2. ServletConfig的常用方法

- **String getInitParameter(“name”)**; 获取当前对象的指定属性值
- **Enumeration getInitParameterNames( )**; 获取所有初始化参数的key，返回一个枚举
- **String getServletName()**; 获取Servlet对象的别名
- **ServletContext getServletContext()**; 获取ServletContext(Servlet上下文)对象

## 4. ServletContext

### 4.0. 简述

- ServletContext 是一个接口，它表示 Servlet 上下文对象
- ServletContext是域对象，代表应用域，整个应用只有一个ServletContext
- ServletContext 是在 web 工程部署启动的时候创建。在 web 工程停止的时候销毁

### 4.1. ServletContext的作用

- 获取全局配置参数(context-param):this.getServletContext().getInitParameter(“key”)
- 保存数据,同一个项目的Servlet共享数据
- 加载配置文件:InputStream in = this.getServletContext().getResourceAsStream(“路径”)
- 获取当前的工程路径和文件的绝对路径:this.getServletContext().getRealPath(“/xx/xxx”)

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws
ServletException, IOException {
// 1、获取 web.xml 中配置的上下文参数 context-param
ServletContext context = getServletConfig().getServletContext();
String username = context.getInitParameter("username");
System.out.println("context-param 参数 username 的值是:" + username);
System.out.println("context-param 参数 password 的值是:" +
context.getInitParameter("password"));
// 2、获取当前的工程路径，格式: /工程路径
System.out.println( "当前工程路径:" + context.getContextPath() );
// 3、获取工程部署后在服务器硬盘上的绝对路径
/**
* 斜杠被服务器解析地址为:http://ip:port/工程名/ 映射到 IDEA 代码的 web 目录<br/>
*/
System.out.println("工程部署的路径是:" + context.getRealPath("/"));
System.out.println("工程下 css 目录的绝对路径是:" + context.getRealPath("/css"));
System.out.println("工程下 imgs 目录 1.jpg 的绝对路径是:" + context.getRealPath("/imgs/1.jpg"));
}
```

- 请求转发:this.getServletContext().getRequestDispatcher(“路径”).forward(req,resp)

### 4.2. ServletContext的常用方法

- void setAttribute(key, value)
- Object getAttributre(key)
- void removeAttribute(key)
- String getInitParameter(String name)
- Enumeration getInitParameterNames();
- String getRealPath(“虚拟路径”):获取绝对路径/真实路径
- String getContextPath():获取工程路径

## 5. 欢迎页面的设置

用户在直接访问webapp时自动定位到的页面即为欢迎页面。欢迎页面可以在web.xml中进行设置。

```xml
<welcome-file-list>
	<welcome-file>login.html</welcome-file>
	<welcome-file>html/welcome.html</welcome-file>
</welcome-file-list>
```

### 欢迎页面的注意点

- 欢迎页面可以设置多个，越靠上优先级越高。
- 欢迎页面可以设置多种，html,jsp,servlet...都可。
- 设置欢迎页面的路径时，**路径开头没有“/”**。

### 欢迎页面的全局配置和局部配置

#### 全局配置

在CATALINA_HOME/conf/web.xml中已经配置好了几种欢迎页面的格式。

#### 局部配置

CATALINA_HOME/webapps/webapp/WEB-INF/web.xml
我们所写的项目的web.xml中的欢迎页面的配置会覆盖全局配置。

## 6. HttpServletRequest

### 6.0. 简述

HttpServletRequest是一个接口，其封装了HTTP请求协议的全部内容，其接口实现类由WEB容器负责实现。
前端传来的数据，会自动封装到request对象中，request对象中有Map集合存放这些数据，Map集合的key是参数的name,value是字符串类型的一维数组。

### 6.1. HttpServletRequest的方法

#### 6.1.0. 获取请求行的相关方法

以URL`http://localhost:8080/request?name=zhangsan&age=17`为例

- **String getMethod()**:获取请求方式 
- **String getRequestURI()**:获取URI。即上面URL中/到问号之前的内容
- **StringBuffer getRequestURL()**:获取URL。即从开始到**问号**之前的内容
- **String getQueryString()**:获取请求参数。即**问号**之后的内容

#### 6.1.1. 获取请求头的相关方法

- **String getHeader(String name)**:获取请求头中的指定内容
- **[Enumeration]<[String]> getHeaderNames()**:获取所有请求头，返回一个枚举
- **int getIntHeader(String name)**:通过一个key获取int类型的值

……

##### 示例

```java
Enumeration<String> headerNames = req.getHeaderNames();
while(headerNames.hasMoreElements()) {
    // 遍历获得key值
	String name = headerNames.nextElement();
    // 通过getHeader("key")获取value值
    String value = req.getHeader(name);
    System.out.println(name + "==" + value);
}
```

#### 6.1.2. 获取请求参数的相关方法

- **String getParameter(String name)**:获取key获取value一维数组的**首元素**
- **String[] getParameterValues(String name)**:通过参数的Map集合的key获取value,返回一个数组
- **Enumeration<String> getParameterNames()**:获取参数的Map对象的所有key
- **Map<String,String[]> getParameterMap()**:获取所有key和value的Map对象(Map<String, String[]>)

#### 6.1.3. 其他方法

- **String getContextPath()**:获取上下文路径
- **String getServletPath()**:获取Servlet路径
- **String getRemoteAddr()**:获取IP地址
- **Object getAttribute(key)**:获取域数据
- **ReuquestDispatcher getRequestDispatcher(String path)**:获取请求转发器
- **Cookie[] getCookies()**:获取所有的Cookie
- **HttpSession getsession()**:获取HttpSession对象,没有则创建一个
- **HttpSession getsession(boolean create)**:获取HttpSession对象，参数为true获取不到时创建，为false时获取不到时返回null
  **未完待续**

## 7. HttpServletResponse

### 7.0. 简述

HttpServletResponse，是接口。每次请求进来，Tomcat服务器都会创建一个Response 对象传递给 Servlet 程序去使用。HttpServletResponse 表示所有响应的信息，我们如果需要设置返回给客户端的信息，都可以通过 HttpServletResponse 对象来进行设置。

### 7.1. 常用方法

- **PrintWriter getWrite()**:获取打印流(字符流)，继而调用**write()**方法往页面写东西。常用于回传字符串
- **ServletOutputStream getOutputStream()**:获取输出流(字节流)，继而调用**write()**方法往页面写东西。常用于下载（传递二进制数据）
  **以上两个流同时只能使用一个**,否则会报错。

```java
public class ResponseIOServlet extends HttpServlet {
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
IOException {
// 要求 ： 往客户端回传 字符串 数据。
PrintWriter writer = resp.getWriter();
writer.write("response's content!!!");
}
}
```

### 7.2. 其他方法

- **void setIntHeader(“refresh”,n)**:n秒后刷新页面
- **void setHeader(“refresh”,“n;url=路径”)**:n秒后跳转页面,可访问外网
- **void addCookie(Cookie cookie)**:添加Cookie

## 8. 乱码问题

### 8.0. 数据保存过程中的乱码

最终保存在数据库的表中的时候出现了乱码。导致乱码的原因有以下两种:

- 数据保存之前，就已经是乱码了，这样保存到数据库中时必然就是乱码了。
- 保存之前数据不是乱码，但由于数据库不支持中文，所以数据保存之后就变成了乱码。

### 8.1.响应乱码

服务器响应浏览器请求，最终显示到网页上的内容出现乱码。
经过了Java程序响应浏览器请求时出现乱码，可以用以下方式解决

#### 8.1.0. 响应乱码解决方式一

```java
resp.setCharacterEncoding("UTF-8");规定服务器的编码格式
resp.setHeader("Content-type","text/html;charset=utf-8");规定浏览器使用的字符编码
```

#### 8.1.1. 响应乱码解决方式二

设置响应的内容类型以及字符的编码方式

```java
resp.setContentType("text/html;charset=utf-8");
```

如果没有经过Java程序，直接访问html页面，可通过在html页面配置相应标签解决

```html
<meta content="text/html;charset=utf-8">
```

### 8.2. 请求乱码

数据从浏览器发送到服务器，在服务器上显示的是乱码

#### 8.2.0. 请求乱码解决方式一

先将浏览器发送过来的数据用ISO-8859-1的方式解码，再给定一种支持简体中文的编码方式重新编码组装。这样既能解决GET请求乱码，又能解决POST请求乱码。

```java
new String(xxx.getBytes("ISO-8859-1","UTF-8"))
```

#### 8.2.1. 请求乱码解决方式二

该方法只准针对POST请求有效，因为它只对请求体进行编码。

```java
request.setCharacterEncoding("字符编码集);
```

#### 8.2.2. 请求乱码解决方式三

专门解决GET请求乱码，因为它只针对请求行编码。
解决方式:修改Tomcat的server.xml文件，(在Contector标签里添加URIEncoding="UTF-8")

## 9. 域对象

- ServletContext,代表**应用域**即application
- HttpSession,代表**session域**
- HttpServletRequest,代表请求域即**request**;
- pageContext(本页面范围,JSP页面独有)

### 9.0. 域对象的范围排序

application > session > request>pageContext

### 9.1. 域对象的数据共享范围

- application跨会话共享数据
- session跨请求共享数据，请求需在同一会话中
- request跨Servlet共享数据，Servlet需在同一请求中
- pageContext,数据在当前页面有效

## 10. 转发和重定向

### 10.0. 转发

```java
req.getRequestDispatcher("/b").forward(request,response);
```

### 10.1. 重定向

```java
resp.sendRedireact(requesy.getContextPath() + "/b"); 
```

### 10.2. 二者的异同点

#### 相同点

都能进行资源的跳转

#### 不同点

- 转发是request对象触发的;重定向是response对象触发的。
- 转发是服务器的行为;重定向是浏览器的行为。
- 转发是**一次请求**，浏览器上地址栏**不会发生变化**;重定向是**二次请求**，浏览器上地址栏会**发生变化**。
- 转发会**保留**请求域中的数据;重定向后，请求域的内容会**改变**。
- 转发是在**项目内部**进行资源的跳转;重定向的路径需要**加上项目的根路径**。
- 转发**不能转到其他应用**，只能在本应用中转发;重定向可以**重定向到项目之外**的应用(外网)。
- 在ServletA中**转发ServletB**执行顺序为:ServletA转发代前的代码->ServletB的代码->ServletA剩余的代码;ServletA中**重定向ServletB**,ServletA中的全部代码会首先执行。