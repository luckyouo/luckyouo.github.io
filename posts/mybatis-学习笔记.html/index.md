# Mybatis 学习笔记


## 前言

MyBatis 是一款优秀的**持久层**框架**，它支持**定制化 SQL**（灵活的修改）**、**存储过程**（函数）以及**高级**映射（javabean和数据库对象的映射->输入映射和输出映射）。MyBatis 避免了几乎所有的 **JDBC** 代码和手动设置参数以及获取结果集。

MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs映射成数据库中的记录。（ORM和持久化）

## 使用流程

### 导包

导入 `mybatis` 和 `mysql` 的依赖。

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.12</version>
    <scope>runtime</scope>
</dependency>
```

### 配置数据库连接

在 `mybatis.xml` 配置配置数据库的参数设置连接。其中 `mapper` 标签处对应的为 `Mapper` 配置文件所在的位置。**（配置文件需要与对应的映射接口处于同一个包下）** ，如果没有配置，则后续 `sql` 语句第一个参数为 `全类名.sql方法` ， 书写更为复杂。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <!--Managed-->
            <transactionManager type="JDBC"/>
            <!--UNPooled-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT"/>
                <property name="username" value="root"/>
                <property name="password" value="admin"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/cskaoyan/dao/UserDao.xml"/>
        <!--<package name="com.cskaoyan.dao"/>-->
    </mappers>
</configuration>
```

### 生成 sqlSession  对象

使用 `构建者` 模式生成 `sqlSession` 对象。

```java
public class MainTest {
    SqlSessionFactory sqlSessionFactory;
    SqlSession sqlSession;

    @Before
    public void init() throws IOException {
        InputStream inputStream = Resources.getResourceAsStream("mybatis.xml");
        //通过建造者创建工厂
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    @After
    public void after(){
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void mytest(){
        sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.queryUserByid(1);
        System.out.println(user);
    }
}
```

### 配置查询语句

在 `UserDao.xml` 配置文件中使用标签配置 `sql` 语句，同名的映射接口配置 `sql` 操作方法，该方法名为 `sql` 语句的 `id` 。

```xml
<mapper namespace="com.cskaoyan.mapper.UserMapper">

    <insert id="insertUser">
        insert into j13_user_t (id, username, password, email) values (#{id}, #{username}, #{password}, #{email})
    </insert>

    <delete id="deleteUser">
        delete from j13_user_t where id = #{id}
    </delete>

    <update id="updateUser">
        update j13_user_t set username = #{username} where id = #{id}
    </update>
</mapper>
```

```java
public interface UserMapper {

    User queryUserByid(int id);

    int insertUser(User user);
    int deleteUser(int id);
    int updateUser(User user);
}
```

`insert` 操作的必须配置 `resultType（或者 resultMap）` ，其他 `sql` 操作没有该参数选项。 

`sql` 语句中传入的参数如果为基本类型时，则任意写，如果为 `javabean` 类型，则使用 `#{类.属性}` 进行选择。可以在调用 `sql` 语句中，使用注解 `@Param` 来限制后续选择，比如

```java
int deleteUser(@Param("id")int id);
```

## 映射关系

### 输入映射

则在配置文件书写 `sql` 语句时，必须使用 `#{id}` 来访问传入参数 `id` 。

如果传入类型为 `pojo` 类型时，没有使用注解时，则使用 `param1` 、`param2` 来选择参数。使用了注解后，则使用注解对应的名字。（使用注解更加清晰），后续使用 `.` 来访问类内的属性。

**使用$符号访问参数时，有注入风险，使用#则mybatis会帮我们进行参数预处理**

### 输出映射

1. 简单类型。查询列必须为单列，才能使用简单类型映射
2. `pojo` 对象映射。在 `ResultType` 字段配置 `pojo` 对象（因为映射接口和配置文件在同一包下，直接使用类小写即可），且查询列名必须与映射的类的属性名一一对应才能映射成功，如果某属性映射不成功，则设置为null。
3. `ResultMap` 对象映射。查询的列名不再需要与对象的属性名一一对应，但是需要通过 `ResultMap` 标签来设置列名和属性名进行映射，其中 `column` 设置查询的结果的列名，`property` 设置属性名。

```xml
<resultMap id="userMap" type="user">
    <result column="id" property="idz"/>
    <result column="username" property="usernames"/>
    <result column="password" property="passwordw"/>
    <result column="email" property="emaill"/>
</resultMap>

<select id="queryUserById" resultMap="userMap">
    select id,username,password,email from j13_user_t where id=#{id}
</select>
```

上述类中的属性名分别为 `idz` 、`usernames` 、`passwordw` 和 `emaill` ，将查询结果与对应属性映射起来。

## 动态 sql 标签

