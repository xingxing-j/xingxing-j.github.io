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

MyBatis是一个支持定制化SQL，存储过程及高级映射的持久层框架。可使用简单的XML或注解用于配置或原始映射，将接口和POJO(Plain Old Java Object,普通Java对象)映射成数据库中的记录。它封装了很多JDBC的细节，使开发者只需关注SQL本身。它使用ORM(Object  Relational  Mapping)思想实现了结果集的封装

## 1. MyBatis主配置文件

SqlMapConfig.xml里有一堆标签。这些标签都在`<configuration>`标签里，**有严格的顺序**

标签顺序：properties，settings，typeAliases，typeHandlers，objectFactory，objectWrapperFactory，reflectFactory，plugins，enviroments，databaseIdProvider，mappers

### 1.0. properties

```xml
<properties resource="jdbcConfig.properties" url="">
    <property name="" value=""/>
    <property name="" value=""/>
</properties>
```

- resource属性配置：按照类路径的写法来写,且该配置文件需在类路径下
- URL属性配置：按照URL的写法写地址
- 可在标签内部配置连接数据库的信息，也可通过${}去引用外部配置文件信息

### 2.0. settings

```xml
<settings>
    <setting name="lazyLoadingEnable" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

- lazyLoadingEnable：延迟加载的全局开关，默认值为：false
- aggressiveLazyLoading：侵入延迟加载，默认值为：true。禁用属性则按需加载
- mapUnderscoreToCamelCase：是否开启自动驼峰命名规则（camel case）映射，默认值：false。数据库字段A_COLUMN 对应 实体类属性aColumn

### 3.0. typeAliases

别名处理器，为实体类配置别名。

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

用来支持多数据库厂商的标签

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

顶级标签，有个属性namespace，对应**接口**的全类名

#### 2.1. resultMap标签

配置**列名**和实体类的**属性名**的对应关系

```xml
<resultMap id="userMap" type="实体类的全限定类名">
    <!-- -主键字段的对应 -->
    <id property="userId" column="id"></id>
    <!-- 非主键字段的对应 -->
    <result property="userName" column="username"></result>
</resultMap>
```

#### 2.2. select标签

- **id**：唯一标识符，对应相关接口的**方法名**


- **parameterType**：传入参数的全类名或别名，可以不写，MyBatis会根据类型推断器(TypeHandler)**自动推断**。

- **resultType**：返回结果的全限定类名或别名。如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。

- **resultMap**：对外部resultMap的引用，resultType 和 resultMap 之间只能同时使用一个。

- **flushCache**：默认值为false。true时，一级缓存、二级缓存都会被清除

  ……


##### 查询所有示例

```xml
<select id="queryAll" reslutType="实体类的全限定类名">
    select * from user;
</select>
```

##### 查询单个示例

```xml
<select id="queryOne" parameterType="INT或Integer或java.lang.Integer" resultType="实体类的全限定类名">
    select * from user;
</select>
```

##### 模糊查询示例

```xml
<select id="queryByName" parameterType="string" resultType="实体类的全限定类名">
    select * from user where username like #{username}; （%在测试类或主要代码类的方法中添加！！）
    或者使用 select * from user where username like '%${value}%';
</select>
```

##### 联合查询(重要)

###### 联合查询之级联属性封装结果集

```xml
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="did" property="dept.id"/>
    <result column="dept_name" property="dept.departmentName"/>
</resultMap>
<select id="getEmpAndDept" resultMap="MyDifEmp">
    SELECT e.id id,e.last_name last_name,e.gender gender,e.d_id d_id,
    d.id did,d.dept_name dept_name FROM tbl_employee e,tbl_dept d
    WHERE e.d_id=d.id AND e.id=#{id}
</select>
```

###### 联合查询之嵌套结果集

使用**association标签**定义关联的单个对象的封装规则

```xml
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp2">
        <id column="id" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="gender" property="gender"/>
		
        <!--  association可以指定联合的javaBean对象
              property="dept"：指定哪个属性是联合的对象
              javaType:指定这个属性对象的类型[不能省略]
         -->
        <association property="dept" javaType="com.atguigu.mybatis.bean.Department">
	           <id column="did" property="id"/>
	           <result column="dept_name" property="departmentName"/>
        </association>
</resultMap>

<select id="getEmpAndDept" resultMap="MyDifEmp">
        SELECT e.id id,e.last_name last_name,e.gender gender,e.d_id d_id,
        d.id did,d.dept_name dept_name FROM tbl_employee e,tbl_dept d
         WHERE e.d_id=d.id AND e.id=#{id}
</select>
```

使用**collection标签**定义关联的集合类型的属性封装规则

```xml
<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDept">
		<id column="did" property="id"/>
		<result column="dept_name" property="departmentName"/>

		<!-- 
			collection定义关联集合类型的属性的封装规则 
			ofType:指定集合里面元素的类型
		-->
		<collection property="emps" ofType="com.atguigu.mybatis.bean.Employee">
			<!-- 定义这个集合中元素的封装规则 -->
			<id column="eid" property="id"/>
			<result column="last_name" property="lastName"/>
			<result column="email" property="email"/>
			<result column="gender" property="gender"/>
		</collection>
	</resultMap>

	<select id="getDeptByIdPlus" resultMap="MyDept">
		SELECT d.id did,d.dept_name dept_name,
				e.id eid,e.last_name last_name,e.email email,e.gender gender
		FROM tbl_dept d
		LEFT JOIN tbl_employee e
		ON d.id=e.d_id
		WHERE d.id=#{id}
</select>
```

##### 分步查询(重要)

###### 使用association标签进行分步查询

```xml
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmpByStep">
	 	<id column="id" property="id"/>
	 	<result column="last_name" property="lastName"/>
	 	<result column="email" property="email"/>
	 	<result column="gender" property="gender"/>

	 	<!-- association定义关联对象的封装规则
	 		select:表明当前属性是调用select指定的方法查出的结果
	 		column:指定将哪一列的值传给这个方法
	 		
	 		流程：使用select指定的方法（传入column指定的这列参数的值）查出对象，
			并封装给property指定的属性
	 	 -->
 		<association property="dept" 
	 		select="com.atguigu.mybatis.dao.DepartmentMapper.getDeptById"
	 		column="d_id">
 		</association>
</resultMap>

<!--  public Employee getEmpByIdStep(Integer id);-->
<select id="getEmpByIdStep" resultMap="MyEmpByStep">
    select * from tbl_employee where id=#{id}
    <if test="_parameter!=null">
        and 1=1
    </if>
</select>
```

##### 使用collection标签进行分步查询

此处待验证

```xml
<!-- collection：分段查询 -->
	<!-- 扩展：多列的值传递过去：
			将多列的值封装map传递；
			column="{key1=column1,key2=column2}"
		fetchType="lazy"：表示使用延迟加载；
				- lazy：延迟
				- eager：立即
	 -->

<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDeptStep">
    <id column="id" property="id"/>
    <id column="dept_name" property="departmentName"/>
    <collection property="emps"
                select="com.atguigu.mybatis.dao.EmployeeMapperPlus.getEmpsByDeptId"
                column="{deptId=id}" fetchType="lazy">
    </collection>
</resultMap>

<!-- public Department getDeptByIdStep(Integer id); -->
<select id="getDeptByIdStep" resultMap="MyDeptStep">
    select id,dept_name from tbl_dept where id=#{id}
</select>
```

#### 2.3. insert标签

```xml
<insert id="addUser" parameterType="参数类型的全限定类名">
      <!-- 配置插入操作后，获取插入数据的主键id
          keyProperty：查出主键值封装给JavaBean的哪个属性值
          order="BEFORE"：当前sql在插入sql之前运行，“AFTER”：当前sql在插入sql之后运行
          resultType：查出的数据的返回值类型
      >
      <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
          select last_insert_id();
      </selectKey>
      insert into user(id,username,address,sex,birhtday) values(#{},#{},#{},#{},#{});
</insert>
```

```xml
<!-- 此处使用useGeneratedKeys属性开启了MySQL的自增主键获取策略；
		keyProperty表示将获取到自增主键封装给JavaBean的哪个属性；
		这里的JavaBean指的是parameterType指定的对象 -->
<insert id="addUser" parameterType="参数类型的全限定类名" useGeneratedKeys="true" keyProperty="id">
    insert into user(username,address,sex,birhtday) values(#{},#{},#{},#{});
</insert>
```

- 采用第一种方式，selectKey标签将获取的id属性封装到int里
- 采用第二种方式，selectKey标签将获取的id属性封装到ParameterType对应的实体类类型里

#### 2.4. update标签

```xml
<update id="updateUser" parameterType="参数类型的全限定类名">
    update 表名 set username=#{username},address=#{address},
    sex=#{sex},birthday=#{birthday} where id=#{uid};
</update>
```

#### 2.5. 鉴别器标签

```xml
<!-- =======================鉴别器============================ -->
	<!-- <discriminator javaType=""></discriminator>
		鉴别器：mybatis可以使用discriminator判断某列的值，然后根据某列的值改变封装行为
		封装Employee：
			如果查出的是女生：就把部门信息查询出来，否则不查询；
			如果是男生，把last_name这一列的值赋值给email;
	 -->
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmpDis">
	 	<id column="id" property="id"/>
	 	<result column="last_name" property="lastName"/>
	 	<result column="email" property="email"/>
	 	<result column="gender" property="gender"/>

	 	<!--
	 		column：指定判定的列名
	 		javaType：列值对应的java类型  -->
	 	<discriminator javaType="string" column="gender">
	 		<!--女生  resultType:指定封装的结果类型；不能缺少。-->

	 		<case value="0" resultType="com.atguigu.mybatis.bean.Employee">
	 			<association property="dept" 
			 		select="com.atguigu.mybatis.dao.DepartmentMapper.getDeptById"
			 		column="d_id">
		 		</association>
	 		</case>

	 		<!--男生 ;如果是男生，把last_name这一列的值赋值给email; -->

	 		<case value="1" resultType="com.atguigu.mybatis.bean.Employee">
		 		<id column="id" property="id"/>
			 	<result column="last_name" property="lastName"/>
			 	<result column="last_name" property="email"/>
			 	<result column="gender" property="gender"/>
	 		</case>
	 	</discriminator>
</resultMap>
```

## 3. 入门案例配置

### 3.0. 入门配置案例一

#### 1、导两个关键的包

mybatis-3.5.3.jar和mysql-connector-java-5.1.47.jar

#### 2、编写MyBatis的主配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--配置的意思-->
<configuration>
    <environments default="development">
        <environment id="development">
            <!--事务管理的配置 -->
            <transactionManager type="JDBC"/>

            <!--数据源的配置 -->
            <dataSource type="POOLED">
                <!--连接信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <!--  此处需注意url的写法，"jdbc:mysql:///mybatistest"
                    也可写 jdbc:mysql://ip:port/mybatistest -->
                <!-- <property name="url" value="jdbc:mysql://localhost:3306/mybatistest"/> -->
                <property name="url" value="jdbc:mysql:///mybatistest"/>
                <property name="username" value="root"/>
                <property name="password" value="rootrr"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--<mapper resource="org/mybatis/example/BlogMapper.xml"/>-->

        <!--不能写.  以文件夹的形式展示映射文件 -->
        <mapper resource="com/javasm/cn/mapper/user.xml"/>
    </mappers>
</configuration>
```

#### 3、编写mapper映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace命名空间   该案例可随意写
select * from 表名 where条件
-->
<mapper namespace="test">
    <select id="findAll"  resultType="com.javasm.cn.entity.User">
        select * from user
    </select>
</mapper>
```

#### 4、编写测试类

```java
public class MyBatisTest {
    @Test
    public void selectAll() throws IOException {
        String resource = "mybatis-config.xml";
        // 加载配置文件
        InputStream inputStream = Resources.getResourceAsStream(resource);

        // 从配置文件的信息中 获取到sqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 获得SqlSession 对象  该对象可以调用映射文件中定义的 各种标签
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<Object> objects = sqlSession.selectList("test.findAll");

        // 关闭资源
        sqlSession.close();
    }
}
```

若想不写DAO的实现类，需遵循以下三点：

- MyBatis中的映射配置文件位置必须和DAO接口的**包结构相同**
- 映射配置文件的mapper标签中的**namespace属性**必须是dao接口的全限定类名
- 映射文件的增删改查的标签中的**id属性取值**必须是dao接口的方法名

## 4. MyBatis的参数处理

### 4.0. 单个参数

MyBatis不会做特殊处理。即`#{参数值}`的形式，`{}`里的参数值写啥都行（自定义类型和自定义包装类型不行）

### 4.1. 多个参数

MyBatis会做特殊处理。多个参数会被封装成一个Map，Map的key为param1到paramN的形式，value就是传入的参数值。可用`#{}`的方式从Map中获取指定的key对应的value值

例：`#{param1}`，`#{param2}`

也可以用**@Param**给方法参数明确命名来指定Map对应的key。**此处待补充示例**

### 4.2. POJO

多个参数正好是业务逻辑的数据模型，可直接传POJO。然后就可以用`#{属性名}`的方式取出传入的POJO对应的属性值。**此处待补充示例**

### 4.3. Map

多个参数不是业务逻辑中的数据模型，没有对应的POJO，不经常使用，为了方便，也可传Map。

### 4.4. TO

多个参数不是业务逻辑中的数据模型，但经常使用，推荐使用TO（Transfer Object）数据传输对象。**不懂啥意思**

## 5. 动态SQL

### 5.0. if和where标签

查询的时候如果某些条件没带sql拼装可能会有问题，解决方法：
1、给where后面加上1=1，以后的条件都and xxx.
2、mybatis使用where标签来将所有的查询条件包括在内。mybatis就会将where标签中拼装的sql，多出来的and或者or去掉。**where标签只会去掉第一个多出来的and或者or**。

```xml
<select id="getEmpsByConditionIf" resultType="com.atguigu.mybatis.bean.Employee">
	 	select * from tbl_employee
	 	<!-- where -->
	 	<where>
		 	<!-- test：判断表达式（OGNL）
		 	OGNL参照PPT或者官方文档。
		 	
		 	遇见特殊符号应该去写转义字符：
		 	&&：
		 	-->
		 	<if test="id!=null">
		 		id=#{id}
		 	</if>
		 	<if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
		 		and last_name like #{lastName}
		 	</if>
		 	<if test="email!=null and email.trim()!=&quot;&quot;">
		 		and email=#{email}
		 	</if> 

		 	<!-- OGNL会进行字符串与数字的转换判断  "0"==0 -->
		 	<if test="gender==0 or gender==1">
		 	 	and gender=#{gender}
		 	</if>
	 	</where>
</select>
```

### 5.1. trim标签

trim标签可自定义字符串的截取规则。

trim标签里的各项属性：

- prefix="前缀''。trim标签体中是整个字符串拼串后的结果。prefix给拼串后的整个字符串加一个前缀 
- prefixOverrides="前缀覆盖"： **去掉整个字符串前面多余的字符**，该属性里面写什么就去除什么
- suffix="后缀"。suffix给拼串后的整个字符串加一个后缀 
- suffixOverrides="后缀覆盖"：**去掉整个字符串后面多余的字符**，该属性里面写什么就去除什么

```xml
<trim prefix="where" suffixOverrides="and">
	 		<if test="id!=null">
		 		id=#{id} and
		 	</if>
		 	<if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
		 		last_name like #{lastName} and
		 	</if>
		 	<if test="email!=null and email.trim()!=&quot;&quot;">
		 		email=#{email} and
		 	</if> 
		 	<!-- OGNL会进行字符串与数字的转换判断  "0"==0 -->
		 	<if test="gender==0 or gender==1">
		 	 	gender=#{gender}
		 	</if>
</trim>
```

### 5.2. choose标签

choose (when, otherwise):分支选择；相当于带了break的swtich-case

```xml
<select id="getEmpsByConditionChoose" resultType="com.atguigu.mybatis.bean.Employee">
        select * from tbl_employee 
        <where>
              <!-- 如果带了id就用id查，如果带了lastName就用lastName查;只会进入其中一个 -->
              <choose>

                      <when test="id!=null">
                                 id=#{id}
                      </when>
                      <when test="lastName!=null">
                                last_name like #{lastName}
                      </when>
                      <when test="email!=null">
                                email = #{email}
                      </when>
                      <otherwise>
                                gender = 0
                      </otherwise>

               </choose>
        </where>
</select>
```

### 5.3. foreach标签

```xml
<select id="getEmpsByConditionForeach" resultType="com.atguigu.mybatis.bean.Employee">
	 	select * from tbl_employee

	 	<!--#{变量名}就能取出变量的值也就是当前遍历出的元素-->
	 	<foreach collection="ids" 
                 item="item_id" 
                 separator="," 
                 open="where id in(" 
                 close=")">
	 		#{item_id}  （此值与item的属性值一致）
	 	</foreach>
</select>
```

foreach标签里的各项属性解释：

- collection(属性)：代表要遍历的集合元素，不要写`#{}`
- open：代表语句的开始部分
- close：代表语句的结束部分
- item：代表遍历集合时生成的变量名
- separator：所用的分隔符
- index:索引。**此处存疑**
  遍历list的时候是index就是索引，item就是当前值
  遍历map的时候index表示的就是map的key，item就是map的值

#### MySQL的批量插入

##### 方式一

```xml
<!--MySQL下批量保存：可以foreach遍历   mysql支持values(),(),()语法-->

<insert id="addEmps">
	 	insert into tbl_employee(
    		<!-- 此处的include标签引用了SQL标签里的SQL语句 -->
	 		<include refid="insertColumn"></include>
	 	) 
		values
		<foreach collection="emps" item="emp" separator=",">
			(#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
		</foreach>
</insert>
```

##### 方式二

**存疑，待验证**

```xml
<!-- 这种方式需要数据库连接属性allowMultiQueries=true；
	 	这种分号分隔多个sql可以用于其他的批量操作（删除，修改） -->

<insert id="addEmps">
	 	<foreach collection="emps" item="emp" separator=";">
	 		insert into tbl_employee(last_name,email,gender,d_id)
	 		values(#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
	 	</foreach>
</insert> 
```

#### Oracle的批量插入

##### 方式一

```xml
<insert id="addEmps" databaseId="oracle">
	 	
    <!-- oracle第一种批量方式,多个insert放在begin - end里面
		例：
 		begin
			    insert into employees(employee_id,last_name,email) 
			    values(employees_seq.nextval,'test_001','test_001@atguigu.com');
			    insert into employees(employee_id,last_name,email) 
			    values(employees_seq.nextval,'test_002','test_002@atguigu.com');
		end;
		-->
    <foreach collection="emps" item="emp" open="begin" close="end;">
    insert into employees(employee_id,last_name,email) 
     values(employees_seq.nextval,#{emp.lastName},#{emp.email});
    </foreach>
	 	
 </insert>
```

##### 方式二

```xml
<insert id="addEmps" databaseId="oracle">
	 	
	 	<!-- oracle第二种批量方式，利用中间表
 			例：
			insert into employees(employee_id,last_name,email)
		       select employees_seq.nextval,lastName,email from(
		              select 'test_a_01' lastName,'test_a_e01' email from dual
		              union
		              select 'test_a_02' lastName,'test_a_e02' email from dual
		              union
		              select 'test_a_03' lastName,'test_a_e03' email from dual
		       )
		-->
	 	insert into employees(
	 		<!-- 引用外部定义的sql -->
	 		<include refid="insertColumn">
	 			<property name="testColomn" value="abc"/>
	 		</include>
	 	)
	 	<foreach collection="emps" item="emp" separator="union"
	 			open="select employees_seq.nextval,lastName,email from("
	 			close=")">
	 			select #{emp.lastName} lastName,#{emp.email} email from dual
	 	</foreach>
</insert>
```

### 5.4. SQL标签 

该标签将经常将要查询的列名，或者插入用的列名抽取出来方便引用。然后在增删改查的标签里用**include标签**来引用已经抽取的sql语句。

```xml
<sql id="insertColumn">
	  		<if test="_databaseId=='oracle'">
	  			employee_id,last_name,email
	  		</if>
	  		<if test="_databaseId=='mysql'">
	  			last_name,email,gender,d_id
	  		</if>
</sql>
```

### 5.5. bind标签

bind标签可以将OGNL表达式的值绑定到一个变量中，方便后来引用这个变量的值。

**待验证和回忆**

```xml
<select id="getEmpsTestInnerParameter" resultType="com.atguigu.mybatis.bean.Employee">
    
    <bind name="_lastName" value="'%'+lastName+'%'"/>

    <if test="_databaseId=='mysql'">
        select * from tbl_employee
        <if test="_parameter!=null">
              where last_name like #{lastName}
        </if>
    </if>
    
    <if test="_databaseId=='oracle'">
          select * from employees
          <if test="_parameter!=null">
                where last_name like #{_parameter.lastName}
          </if>
     </if>

 </select>
```

### 5.6. 动态SQL内置对象

- **_parameter**：代表整个参数
  - 如果只有单个参数，那么**_parameter**就是这个参数
  - 如果有多个参数，这些参数会被封装为一个map。**_parameter**就是代表这个map
- **_databaseId**：如果配置了databaseIdProvider标签的话。 **_databaseId**就是代表当前数据库的别名

```xml
<select id="getEmpsTestInnerParameter" resultType="com.atguigu.mybatis.bean.Employee">

	  		<if test="_databaseId=='mysql'">
	  			select * from tbl_employee
	  			<if test="_parameter!=null">
	  				where last_name like #{lastName}
	  			</if>
	  		</if>
	  		<if test="_databaseId=='oracle'">
	  			select * from employees
	  			<if test="_parameter!=null">
	  				where last_name like #{_parameter.lastName}
	  			</if>
	  		</if>
</select>
```

## 6. MyBatis的缓存

MyBatis中默认定义了两级缓存。

### 6.0. 一级缓存

SqlSession级别的缓存，也称为**本地缓存**。默认开启。

#### 一级缓存失效的情况

- SSM整合后，一级缓存就失效了。大概，待验证
- sqlSession不同
- sqlSession相同，查询条件不同
- sqlSession相同，两次查询之间执行了增删改操作（第二次查询可能对当前数据有影响）
- sqlSession相同，手动清除了一级缓存

### 6.1. 二级缓存

基于namespace级别的缓存，也称为**全局缓存**。需手动开启和配置。也可通过Cache接口自定义二级缓存。

#### 二级缓存的工作机制简述

1、一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中

2、如果会话关闭，一级缓存中的数据会被保存到二级缓存中。新的会话查询信息，就可以参照二级缓存中的内容

3、sqlSession===EmployeeMapper==>Employee DepartmentMapper===>Department

#### 效果

**待补充验证**

不同namespace查出的数据会放在自己对应的缓存中（map）

数据会从二级缓存中获取。查出的数据都会被默认先放在一级缓存中。

只有会话提交或者关闭以后，一级缓存中的数据才会转移到二级缓存中。

#### 如何开启二级缓存

- 1、MyBatis的配置文件中**开启全局二级缓存配置**：\<setting name="cacheEnabled" value="true"/>
- 2、在对应的mapper映射文件中配置**\<cache/>标签**来使用二级缓存

```xml
<cache eviction="FIFO" 
       flushInterval="60000" 
       readOnly="false" 
       size="1024"></cache>
```

cache相关属性解释：

- eviction:缓存的回收策略(默认的是 **LRU**)：
  - LRU – 最近最少使用的：移除最长时间不被使用的对象。
  - FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
  - SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
  -  WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
- flushInterval：缓存刷新间隔。缓存多长时间清空一次，默认不清空，设置单位为毫秒值
- readOnly:是否只读。
  - true：只读。mybatis认为所有从缓存中获取数据的操作都是只读操作，不会修改数据。 mybatis为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，速度快
  - false：非只读。mybatis觉得获取的数据可能会被修改。mybatis会利用序列化&反序列的技术克隆一份新的数据给你。安全，速度慢
- size：缓存大小
- type=""：指定自定义缓存的全类名，填写已实现Cache接口的类的全类名即可。
- 3、**POJO需要实现序列化接口**

### 6.2. 缓存的相关配置和属性

- cacheEnabled=true/false。false时关闭缓存（一级缓存一直可用的，二级缓存关闭)
- select标签的useCache="true/false"。false时不使用缓存（一级缓存依然使用，二级缓存不使用）
- 增删改标签的**flushCache="true"**（增删改执行完成后就会清除一级和二级缓存）
- **sqlSession.clearCache()**只是清除当前session的一级缓存
	 **待验证**？？localCacheScope：本地缓存作用域（一级缓存SESSION）当前会话的所有数据保存在会话缓存中                                                                                      STATEMENT：可以禁用一级缓存；		

### 6.3. 第三方缓存配置

导入第三方缓存包，再导入与第三方缓存整合的适配包，官方有。

```xml
mapper.xml中使用自定义缓存
<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>

其它mapper.xml文件也可引用：
<!-- 引用缓存：namespace：指定和哪个名称空间下的缓存一样 -->
<cache-ref namespace="com.atguigu.mybatis.dao.EmployeeMapper"/>
```



