---
title: SSM整合之jar包版
date: 2020-05-20 11:19:03
categories: 后端学习
tags:
- [SSM整合]
---

以导jar包的形式，不以Maven的方式构建，进行的SSM整合

<!-- more -->

## 0. 导包

### Spring4个核心包和1个日志包

- **spring-beans-5.1.9.RELEASE.jar**
- **spring-context-5.1.9.RELEASE.jar**
- **spring-core-5.1.9.RELEASE.jar**
- **spring-expression-5.1.9.RELEASE.jar**
- **spring-jcl-5.1.9.RELEASE.jar**和**commons-logging-1.2.jar**二选一

### 导入AOP和AspecJ相关的包

- **spring-aop-5.1.9.RELEASE.jar**
- **aspectjweaver-1.9.4.jar**


### 导入Spring对持久层的支持包

- **spring-tx-5.1.9.RELEASE.jar**
- **spring-jdbc-5.1.9.RELEASE.jar**

### 导入数据源相关的包(可选)

- **druid-1.1.10.jar**(可选)，如果不用druid数据源，也可在Spring配置文件中使用Spring或Mybatis提供的数据源

### 导入SpringMVC相关的包

- **spring-web-5.1.9.RELEASE.jar**
- **spring-webmvc-5.1.9.RELEASE.jar**

### 导入spring-mybatis整合包

- **mybatis-spring-1.3.2.jar**

### 导入jackson相关的包

- **jackson-annotations-2.9.8.jar**
- **jackson-core-2.9.8.jar**
- **jackson-databind-2.9.8.jar**
- **jackson-datatype-jsr310-2.9.8.jar**


### 导入MyBatis相关的包

- 包含mybatis的包(**必选**)
  - **mybatis-3.5.3.jar**
- mybatis包中的lib目录下的依赖包(**可选**)
  - ant-1.10.3.jar
  - ant-launcher-1.10.3.jar
  - asm-7.0.jar
  - cglib-3.2.10.jar
  - javassist-3.24.1-GA.jar
  - log4j-1.2.17.jar
  - log4j-api-2.11.2.jar
  - log4j-core-2.11.2.jar
  - ognl-3.2.10.jar
  - slf4j-api-1.7.26.jar
  - slf4j-log4j12-1.7.26.jar

### 导入相关数据库驱动包

- **mysql-connector-java-5.1.47.jar**

## 1. 配置文件

### 数据库配置文件

**db.properties**，该项配置文件的**URL**是否可行还需具体验证

```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/数据库名称?useSSL=false&useUnicode=true&characterEncoding=utf8
jdbc.username=用户名
jdbc.password=密码
```

#### log4j的配置文件

```properties
### 设置###
log4j.rootLogger = debug,stdout,D,E
### 输出信息到控制抬 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

### 输出DEBUG 级别以上的日志到=E://logs/error.log ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = E://logs/log.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### 输出ERROR 级别以上的日志到=E://logs/error.log ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File =E://logs/error.log 
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR 
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

### MyBatis的配置文件

由于MyBatis配置文件里的相关配置，都可以在Spring的配置文件中配置，所有下面的MyBatis的配置文件可以被取代了。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--配置标签-->
<configuration>
    <properties resource=""/>
    <environments default="development">
        <environment id="development">
            <!--事务管理的配置 -->
            <transactionManager type="JDBC"/>
            <!--数据源的配置 -->
            <dataSource type="POOLED">
                <!--连接信息-->
                <property name="driver" value="${jdbc.driver}"/>
                <!--  此处需注意url的写法，"jdbc:mysql:///xxx"
                    也可写 jdbc:mysql://ip:port/xxx -->
                <!-- <property name="url" value="jdbc:mysql://localhost:3306/xxx"/> -->
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!--不能写点，以文件夹的形式展示映射文件 -->
        <mapper resource=""/>
    </mappers>
</configuration>
```

### Spring配置文件

文件名一般叫`applicationcontext.xml`，也可自定义文件名。该Spring配置文件中集合了包扫描、事务、AOP、数据源等等配置。也可将该配置文件分为多个配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 配置包扫描，只扫service层的包 -->
    <context:component-scan base-package="com.xxx.cn.service"/>

    <!-- 加载数据库配置文件 -->
    <context:property-placeholder location="classpath:db.properties"/>

        <!-- 配置数据源，这里配的数据源可以是MyBatis的PooledDataSource，也可以是Spring的DrivermanagerDataSource，这里用的是Druid数据源 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 加载MyBatis的配置文件，因为MyBatis的配置Spring里也可以配，所以没用了 -->
        <!-- <property name="configLocation" value="classpath:mybatis-config.xml"/> -->
    </bean>

    <!-- 加载MyBatis的映射文件 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.xxx.cn.mapper"/>
    </bean>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 引入外部的数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 开启事务注解驱动 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

### SpringMVC配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 配置包扫描，仅扫描controller包 -->
    <context:component-scan base-package="com.xxx.cn.controller"/>

    <!-- mvc:annotation-driven标签会默认会帮我们注册默认处理请求，参数和返回值的类 -->
    <mvc:annotation-driven/>
</beans>
```

### web.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationcontext.xml</param-value>
    </context-param>
    
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    
    <!-- 配置SpringMVC的前端控制器 -->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

## 2、编写其他类测试

### 实体类

```java
public class User {
    private String username;
    private String password;
	// getter、setter、toString方法略   
}
```

### mapper接口

这里不需要加**@Repository**注解，因为在Spring的配置文件中专门配置了一个扫描MyBatis映射文件的类。

```java
public interface IUserMapper {
    @Select("SELECT * FROM admin")
    List<User> queryAllUser ();
}
```

### Service接口及实现类

这里不给接口加**@Service**注解，而是给它的实现类加。

#### 接口

```java
public interface IUserService {
    public List<User> queryAll ();
}
```

#### 实现类

```java
@Service // 将该类加入到IoC容器中
@Transactional //  在Service层加注解进行事务控制。默认发生运行时异常回滚，发生编译时异常不回滚。
public class UserServiceImpl implements IUserService {
    @Autowired // 此处写的是接口类型，将其自动装配
    private IUserMapper iUserMapper;
    @Override
    public List<User> queryAll() {
        List<User> users = iUserMapper.queryAllUser();
        return users;
    }
}
```

### Controller类

```java
@Controller // 将该类加入到IoC容器中
@RequestMapping("/user")
public class UserController {
    @Autowired// 此处写的是接口类型，将其自动装配
    private IUserService iUserService;

    @GetMapping("/queryAll")
    public ResponseEntity queryAll () {
        List<User> users = iUserService.queryAll();
        return ResponseEntity.ok(users);
    }
}
```

之后开启tomcat进行测试即可

## 注意点

Spring的配置文件，可按各自功能分成多个。

### spring-mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 加载数据库配置文件 -->
    <context:property-placeholder location="classpath:db.properties"/>

    <!-- 配置数据源，这里配的数据源可以是MyBatis的PooledDataSource，也可以是Spring的DrivermanagerDataSource，这里用的是Druid数据源 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 加载MyBatis的配置文件，因为MyBatis的配置Spring里也可以配，所以没用了 -->
        <!-- <property name="configLocation" value="classpath:mybatis-config.xml"/> -->
    </bean>

    <!-- 加载MyBatis的映射文件 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.xxx.cn.mapper"/>
    </bean>
</beans>
```

### spring-service.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置包扫描，只扫service层的包 -->
    <context:component-scan base-package="com.xxx.cn.service"/>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 引入外部的数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 开启事务注解驱动 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

将Spring的配置文件分为以上两个后，可以选择在原配置文件中用**import**标签引入它们，这样就不用修改web.xml文件了。例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <import resource="spring-mapper.xml"/>
    <import resource="spring-service.xml"/>
</beans>
```

或者启用原配置文件，修改web.xml文件也可。例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <!-- 这里的值也可写成classpath:spring-mapper.xml,classpath:spring-service.xml -->
        <param-value>classpath:spring-*.xml</param-value>
    </context-param>
    
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    
    <!-- 配置SpringMVC的前端控制器 -->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
</web-app>
```









