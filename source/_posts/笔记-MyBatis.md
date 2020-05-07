---
title: 笔记-MyBatis
date: 2020-04-20 19:45:55
categories: 后端学习
tags:
- [MyBatis]
- [框架]
---

施工中

<!-- more -->

# MyBatis

## 0. 简介

> - MyBatis是一个支持定制化SQL，存储过程及高级映射的持久层框架
> - 可使用简单的XML或注解用于配置或原始映射，将接口和POJO(Plain Old Java Object,普通Java对象)映射成数据库中的记录
> - 它封装了很多JDBC的细节，使开发者只需关注SQL本身。它使用ORM(Object  Relational  Mapping)思想实现了结果集的封装

## 1. MyBatis主配置文件

> SqlMapConfig.xml里有一堆标签。这些标签都在`<configuration>`标签里，有严格的顺序
>
> 标签顺序：properties，settings，typeAliases，typeHandlers，objectFactory，objectWrapperFactory，reflectFactory，plugins，enviroments，databaseIdProvider，mappers

### 1.0. properties

```xml
<properties resource="jdbcConfig.properties" url="">
    <property name="" value=""/>
    <property name="" value=""/>
</properties>
```

> - resource属性配置：按照类路径的写法来写,且该配置文件需在类路径下
> - URL属性配置：按照URL的写法写地址
> - 可在标签内部配置连接数据库的信息，也可通过${}去引用外部配置文件信息

### 2.0. settings

```xml
<settings>
    <setting name="lazyLoadingEnable" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

> - lazyLoadingEnable：延迟加载的全局开关，默认值为：false
> - aggressiveLazyLoading：侵入延迟加载，默认值为：true。禁用属性则按需加载
> - mapUnderscoreToCamelCase：是否开启自动驼峰命名规则（camel case）映射，默认值：false。数据库字段A_COLUMN 对应 实体类属性aColumn

### 3.0. typeAliases

> 别名处理器，为实体类配置别名

```xml
<typeAliases>
    <!-- type为要配置的全限定类名，光配这一个属性的话默认别名为类名的小写  -->
    <typeAlias type="xxx.xx.User" alias="user"/>
    
    <!-- 用于指定要配置别名的包，指定后该包下的实体类都会注册别名，且别名就是类名，不区分大小写-->
    <package name="xxx.xx.entity"/>
</typeAliases>
```

### 4.0. envrionments

```xml
<!-- default：指定使用某种环境，达到快速切换的目的，填的是envrioment标签的id属性 -->
<environments default="mysql">
	<!--MySQL配置环境-->
	<environment id="mysql">
        
	<!-- 配置事务管理器 
		 type属性：事务管理器类型 
			JDBC（JdbcTransactionFactory）；MANAGED（ManagedTransactionFactory）；自定义事务管理				器，需实现TransactionFactory接口，type为全限定类名
		-->
		<transactionManager type="JDBC"/>
		<!-- 配置数据源连接池
			 type属性：
				POOLED：采用传统的javax.sql.DataSource规范中的连接池，MyBatis中有针对其规范的实现
				UNPOOLED：采用传统的获取连接的方式，虽然也实现了javax.sql.DataSource接口，但没有使用						  池的思想
				JNDI：采用服务器提供的JNDI技术实现，来获取DataSource对象。注：如果不是Web或Maven的war					   工程，则不能使用
		-->
		<dataSource type="POOLED">
			<!--配置连接数据库的4个基本信息-->
			<property name="driver" value="${}"/>
			<property name="url" value="${}"/>
			<property name="username" value="${}"/>
			<property name="password" value="${}"/>
		</dataSource>
	</environment>
</environments>
```

### 5.0. databaseIdProvider

> 用来支持多数据库厂商的标签

```xml
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver"/>
  <property name="DB2" value="db2"/>
  <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

### 6.0. mappers

```xml
<mappers>
	<!-- 使用相对于类路径的资源引用 -->
	<mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
	<!-- 使用完全限定资源定位符（URL） -->
	<mapper url="file:///var/mappers/AuthorMapper.xml"/>
  	<!-- 使用映射器接口实现类的完全限定类名 -->
    <mapper class="org.mybatis.builder.AuthorMapper"/>
    <!-- 将包内的映射器接口实现全部注册为映射器 -->
    <package name="org.mybatis.builder"/>
</mappers>
```

## 2. xml映射文件

### 2.0. mapper标签

> 顶级标签，有个属性namespace，对应**接口**的全类名

### 2.1. resultMap标签

> 配置**列名**和实体类的**属性名**的对应关系

```xml
<resultMap id="userMap" type="实体类的全限定类名">
	<!-- -主键字段的对应 ->
    <id property="userId" column="id"></id>
    <!-- 非主键字段的对应>
    <result property="userName" column="username"></result>    
</resultMap>
```



### 2.2. select标签

> - **id**：唯一标识符，对应相关接口的**方法名**
> - **parameterType**：传入参数的全类名或别名，可以不写，MyBatis会根据类型推断器(TypeHandler)自动推断。
> - **resultType**：返回结果的全限定类名或别名。如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。
> - **resultMap**：对外部resultMap的引用，resultType 和 resultMap 之间只能同时使用一个。
> - **flushCache**：默认值为false。true时，一级缓存、二级缓存都会被清除
>
> ……

