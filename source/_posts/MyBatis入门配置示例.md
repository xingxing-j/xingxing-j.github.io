---
title: MyBatis入门配置示例
date: 2020-04-20 19:46:55
categories: 入门配置示例
tags:
- [MyBatis]
- [配置案例]
---

MyBatis的几个入门示例

<!-- more -->

### 示例一

该示例不使用mapper代理的方式，也不写DAO接口和DAO实现类

#### 1、导包

导两个关键的包，**mybatis-3.5.3.jar**和**mysql-connector-java-5.1.47.jar**

#### 2、编写MyBatis主配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 配置标签 -->
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
                <property name="url" value="jdbc:mysql:///mybatistest?useSSL=false"/>
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
<mapper namespace="test">
    <select id="findAll"  resultType="com.javasm.cn.entity.User">
        select * from user
    </select>

    <select id="findOne" resultType="com.javasm.cn.entity.User">
        select * from user where uage=#{uage}
    </select>

    <!-- 此处#{}里的值必须要和传进来的实体类的属性名一致，包括大小写
         insert标签可不用写parameterType属性
    -->
    <insert id="insertOne">
        INSERT INTO user (uname, uage, ubirthday)   VALUES(#{uName}, #{uAge}, #{uBirthday})
    </insert>

    <delete id="deleteOne">
        DELETE FROM user WHERE uid=#{id}
    </delete>

    <update id="updateOne">
        UPDATE user SET uname=#{uName} WHERE uid=#{uId}
    </update>
</mapper>
```

#### 4、编写实体类

```java
public class User {
    private Integer uId;
    private String uName;
    private Integer uAge;
    private Date uBirthday;

	// setter、getter、toString方法略

    public User() {
    }

    public User(String uName, Integer uAge, Date uBirthday) {
        this.uName = uName;
        this.uAge = uAge;
        this.uBirthday = uBirthday;
    }
}
```

#### 5、编写测试类测试

```java
public class MyBatisTest {
    private SqlSession sqlSession;

    @Before
    public void beforeTest () throws IOException {
        String resource = "mybatis-config.xml";
        // 加载配置文件
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 从配置文件的信息中 获取到sqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 获得SqlSession 对象  该对象可以调用映射文件中定义的 各种标签
        sqlSession = sqlSessionFactory.openSession();
    }

    @Test
    public void selectAll () throws IOException {
        // 查询所有
        List<Object> objects = sqlSession.selectList("test.findAll");
        System.out.println(objects);
    }


    public User selectOne (Integer uAge) throws IOException {
        // 查询单个
        User o = (User)sqlSession.selectOne("test.findOne", uAge);
        System.out.println(o);
        return o;
    }

    @Test
    public void addOne () {
        // 插入单条数据
        User user = new User("王三", 15, new Date());
        sqlSession.insert("test.insertOne", user);
    }

    @Test
    public void deleteOne () {
        // 删除单条数据
        sqlSession.delete("test.deleteOne", 4);
    }

    @Test
    public void updateOne () throws IOException {
        // 更新单条数据
        User user = selectOne(14);
        user.setuName("更新后的王二");
        sqlSession.update("test.updateOne", user );
    }

    @After
    public void afterTest () {
        sqlSession.commit();
        // 关闭资源
        sqlSession.close();
    }
}
```

### 示例二

该示例不使用mapper代理的方式，编写mapper接口及其实现类。

#### 1、导包

导两个关键的包，**mybatis-3.5.3.jar**和**mysql-connector-java-5.1.47.jar**

#### 2、编写MyBatis主配置文件

**注意**：这里的配置文件中的**mapper标签**里使用了resource属性来引入外部的mapper映射文件，所以，对mapper接口和mapper映射文件二者位置的关系不做要求。如果是使用class、url或package属性引入的话，这两个文件，必须要在同一目录结构下。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--配置标签-->
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
                <property name="url" value="jdbc:mysql:///mybatistest?useSSL=false"/>
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
<mapper namespace="test">
    <select id="findAll"  resultType="com.javasm.cn.entity.User">
        select * from user
    </select>

    <select id="findOne" resultType="com.javasm.cn.entity.User">
        select * from user where uage=#{uage}
    </select>

    <!-- 此处#{}里的值必须要和传进来的实体类的属性名一致，包括大小写
         insert标签可不用写parameterType属性
    -->
    <insert id="insertOne">
        INSERT INTO user (uname, uage, ubirthday)   VALUES(#{uName}, #{uAge}, #{uBirthday})
    </insert>

    <delete id="deleteOne">
        DELETE FROM user WHERE uid=#{id}
    </delete>

    <update id="updateOne">
        UPDATE user SET uname=#{uName} WHERE uid=#{uId}
    </update>
</mapper>
```

#### 4、编写mapper接口

mapper、dao、handler，都一个东西

```java
public interface IUserDAO {

    List<User> selectAll ();

    User selectOne (Integer uAge);

    int insertOne (User user);

    int updateOne (User user);

    int deleteOne (Integer uId);
}
```

#### 5、编写mapper实现类

```java
public class UserDAOImpl implements IUserDAO {
    private SqlSession sqlSession;

    private void init () throws IOException {
        String resource = "mybatis-config.xml";
        // 加载配置文件
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 从配置文件的信息中 获取到sqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 获得SqlSession 对象  该对象可以调用映射文件中定义的 各种标签
        sqlSession = sqlSessionFactory.openSession(true);
    }

    public UserDAOImpl() throws IOException {
        init();
    }

    public void release () {
        sqlSession.close();
    }

    @Override
    public List<User> selectAll() {
        List<User> users = sqlSession.selectList("test.findAll");
        return users;
    }

    @Override
    public User selectOne(Integer uAge) {
        return (User)sqlSession.selectOne("test.findOne", uAge);
    }

    @Override
    public int insertOne(User user) {
        return sqlSession.insert("test.insertOne", user);
    }

    @Override
    public int updateOne(User user) {
        return sqlSession.update("test.updateOne", user);
    }

    @Override
    public int deleteOne(Integer uId) {
        return sqlSession.delete("test.deleteOne", uId);
    }
}
```

#### 6、编写实体类

```java
public class User {
    private Integer uId;
    private String uName;
    private Integer uAge;
    private Date uBirthday;

	// setter、getter、toString方法略

    public User() {
    }

    public User(String uName, Integer uAge, Date uBirthday) {
        this.uName = uName;
        this.uAge = uAge;
        this.uBirthday = uBirthday;
    }
}
```

#### 7、编写测试类测试

```java
public class MyBatisDAOTest {
    private UserDAOImpl userDAO = new UserDAOImpl();

    public MyBatisDAOTest() throws IOException {
    }

    @After
    public void release () {
        userDAO.release();
    }

    @Test
    public void testSelectAll () {
        List<User> users = userDAO.selectAll();
    }

    @Test
    public void testSelectOne () {
        User user = userDAO.selectOne(14);
    }

    @Test
    public void testInsertOne () {
        User user = new User("王四", 8, new Date());
        userDAO.insertOne(user);
    }

    @Test
    public void testUpdateOne () {
        User user = userDAO.selectOne(14);
        user.setuName("改后的王");
        userDAO.updateOne(user);
    }

    @Test
    public void testDeleteOne () {
        userDAO.deleteOne(1);
    }
}
```

### 示例三

该示例使用了mapper代理的方式，免去了编写mapper接口的实现类。

#### 1、导包

导两个关键的包，**mybatis-3.5.3.jar**和**mysql-connector-java-5.1.47.jar**

#### 2、编写MyBatis主配置文件

**注意**：这里的配置文件中的**mapper标签**里使用了resource属性来引入外部的mapper映射文件，所以，对mapper接口和mapper映射文件二者位置的关系不做要求。**如果是使用class、url或package属性引入的话，这两个文件，必须要在同一目录结构下**。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--配置标签-->
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
                <property name="url" value="jdbc:mysql:///mybatistest?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="rootrr"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--<mapper resource="org/mybatis/example/BlogMapper.xml"/>-->

        <!--不能写点，以文件夹的形式展示映射文件 -->
        <mapper resource="com/javasm/cn/xml/usermapper.xml"/>
    </mappers>
</configuration>
```

#### 3、编写mapper映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.javasm.cn.mapper.IUserMapper">
    <select id="selectAll"  resultType="com.javasm.cn.entity.User">
        select * from user
    </select>

    <select id="selectOneByAge" resultType="com.javasm.cn.entity.User">
        select * from user where uage=#{uage}
    </select>

    <!-- 此处#{}里的值必须要传进来的实体类的属性名一致，包括大小写
         insert标签可不用写parameterType属性
    -->
    <insert id="insertOne">
        INSERT INTO user (uname, uage, ubirthday)   VALUES(#{uName}, #{uAge}, #{uBirthday})
    </insert>

    <update id="updateOne">
        UPDATE user SET uname=#{uName} WHERE uid=#{uId}
    </update>

    <delete id="deleteOne">
        DELETE FROM user WHERE uid=#{id}
    </delete>

</mapper>
```

#### 4、编写mapper接口

```java
public interface IUserMapper {
    List<User> selectAll ();

    User selectOneByAge (Integer uAge);

    int insertOne (User user);

    int updateOne (User user);

    int deleteOne (Integer uId);
}
```

#### 5、编写实体类

```java
public class User {
    private Integer uId;
    private String uName;
    private Integer uAge;
    private Date uBirthday;

	// setter、getter、toString方法略

    public User() {
    }

    public User(String uName, Integer uAge, Date uBirthday) {
        this.uName = uName;
        this.uAge = uAge;
        this.uBirthday = uBirthday;
    }
}
```

#### 6、编写测试类测试

```java
public class MyBatisTestByProxy {
    private SqlSession sqlSession;
    private IUserMapper mapper;
    @Before
    public void beforeTest () throws IOException {
        String config = "mybatis-config.xml";
        InputStream in = Resources.getResourceAsStream(config);
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(in);
        sqlSession = build.openSession(true);
        mapper = sqlSession.getMapper(IUserMapper.class);

    }

    @After
    public void afterTest () {
        sqlSession.close();
    }
    @Test
    public void testSelectAll () {
        List<User> users = mapper.selectAll();
        System.out.println(users);
    }

    @Test
    public void testSelectOneByAge () {
        User user = mapper.selectOneByAge(14);
        System.out.println(user);
    }

    @Test
    public void testInsertOne () {
        User user = new User("王六", 22, new Date());
        mapper.insertOne(user);
    }

    @Test
    public void testUpdateOne () {
        User user = mapper.selectOneByAge(14);
        user.setuName("又改回来的王");
        mapper.updateOne(user);
    }

    @Test
    public void testDeleteOne () {
        mapper.deleteOne(7);
    }
}
```

