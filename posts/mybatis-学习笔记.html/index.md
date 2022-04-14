# Mybatis 学习笔记


## 前言

MyBatis 是一款优秀的**持久层**框架**，它支持**定制化 SQL**（灵活的修改）**、**存储过程**（函数）以及**高级**映射（javabean和数据库对象的映射->输入映射和输出映射）。MyBatis 避免了几乎所有的 **JDBC** 代码和手动设置参数以及获取结果集。

MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 `POJO` 映射成数据库中的记录。（ORM和持久化）

{{< figure src="/images/mybatis.png" title="mybatis 流程图" >}}

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

在 `mybatis.xml` 配置配置数据库的参数设置连接。主要配置内容为数据库驱动jdbc `driver` 、 数据库的连接信息和查询映射文件。

映射文件配置有两种方式：

- 配置 `mapper` 标签处对应的为 `Mapper` 配置文件所在的位置`rsource` 。**（配置文件需要与对应的映射接口处于同一个包下）** ，如果没有配置，则后续 `sql` 语句第一个参数为 `全类名.sql方法` ， 书写较为复杂。
- 在 `mapper` 标签内配置 `package` 参数，该参数表示配置文件所在的包目录。（主要使用）

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

1. 简单类型。对应简单基本类型来说，如果没有配置注解 `@Param` ，则使用名称小写即可（内部以实现映射）。使用了注解，则根据注解使用。

```xml
<update id="updateUser">
    update j13_user_t set username = #{username} where id = #{id}
</update>
```

2. `POJO` ：没有使用注解时，使用类的小写即可（其中配置文件必须与类在同一个包下）。使用注解时，根据注解使用。如果传入类型为多个 `pojo` 类型时，且没有使用注解时，则使用 `param1` 、`param2` 来选择参数。使用了注解后，则使用注解对应的名字。（使用注解更加清晰），后续使用 `.` 来访问类内的属性。

```xml
<update id="updateUser">
    update j13_user_t set username = #{username} where id = #{param1.id}
</update>
```

```xml
<update id="updateUser">
    update j13_user_t set username = #{username} where id = #{user.id}
</update>
```

3. `Map` ：若只有一个传入参数，直接使用 `Key` 获取 `Value` 。若存在多个传入参数，则使用 `param1` 、`param2` 来选择。使用注解后根据注解使用。

```xml
<update id="updateUser">
    update j13_user_t set username = #{username} where id = #{param1.key}
</update>
```

```xml
<update id="updateUser">
    update j13_user_t set username = #{username} where id = #{map.key}
</update>
```

**凡是使用了注解 `@Param` ，都根据注解进行匹配，尽量使用注解，使用注解可以提高代码可读性。**

**使用$符号访问参数时，有注入风险，使用#则mybatis会帮我们进行参数预处理**

### 输出映射

1. 简单类型。查询列必须为单列，才能使用简单类型映射

```xml
<select id="selectUser" resultType="string">
    select id from j13_user_t where username = #{user.username}
</update>
```

2. `pojo` 对象映射。在 `ResultType` 字段配置 `pojo` 对象（因为映射接口和配置文件在同一包下，直接使用类小写即可），且查询列名必须与映射的类的属性名一一对应才能映射成功，如果某属性映射不成功，则设置为null。`resultType` 为类名的小写（配置文件必须与类在一个包内）。

```java
public class User{
    int id;
    String username;
    String password;
    String email;
}
```

```xml
<select id="selectUser" resultType="user">
    select id,username,password,email from j13_user_t where username = #{user.username}
</update>
```

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

常用的标签有 `where` 、 `if` 、`sql` 和 `foreach` 。标签的主要作用是将 `sql` 语句分成若干段，可以使用 `inclde` 标签和 `refid` 属性引用 `sql` 语句段，从而代码重用，减少冗余代码。

### `where` 标签

```xml
<select id="selectUser" resultType="string">
    select id from j13_user_t 
    <where>
        username = #{user.username}
    </where>
</update>
```

### `if` 标签。

需要注意的是除了第一个 `if` 标签，后续 `if` 标签都需要添加 `and` 或者 `or` ，来表示层次关系。

```xml
<select id="selectUser" resultType="string">
    select id from j13_user_t 
    <where>
        <if test="username != null">
        	username = #{username}
        </if>
        <if test="age != 0">
        	and age = #{age}
        </if>
    </where>
</update>
```

### `sql` 标签。

主要是用作片段，给其他语句 `include` 使用。

```xml
<sql id="selectAll">
	select id,username,password,email from j13_user_t
</sql>
```

### `foreach` 标签。

用来遍历链表或者数组。

- 未使用注解的数组。`collection` 使用 `array` , `item` 表示每次迭代的对象，`separator` 表示每次迭代后的连接符号，`index` 表示索引，从 `0` 开始。

```xml
<insert id="insertArray">
	insert into j13_user_t (id,username,password,email) values
    <foreach collection="array" item="user" sepatrtor="," index="index">
    	(#{user.id}, #{user.username},#{user.password},#{user.email})
	</foreach>
</insert>
```

- 未使用注解的List。`collection` 使用 `list` 。`open` 和 `close` 表示迭代后的字符串左右两边的符号。若不使用的这两个属性，需要手动增加两个括号，代码不够优雅。

```xml
<insert id="selectUserById" resultType="user">
	select id,username,password,email from j13_uset_t
    <where>
        id in 
        <foreach collection="list" item="userid" open="(" close")">
    		#(userid)
		</foreach>
    </where>    
</insert>
```

- 使用注解的数组和List。

```xml
<insert id="insertArray">
	insert into j13_user_t (id,username,password,email) values
    <foreach collection="users" item="user" sepatrtor=",">
    	(#{user.id}, #{user.username},#{user.password},#{user.email})
	</foreach>
</insert>
```

```xml
<insert id="selectUserById" resultType="user">
	select id,username,password,email from j13_uset_t
    <where>
        id in 
        <foreach collection="ids" item="id" open="(" close")">
    		#(id)
		</foreach>
    </where>    
</insert>
```

## Mybatis 的优化

### 懒加载

根据数据的使用情况，按需加载  sql 语句。当某个  sql 语句执行完成后，没有书用到其中的数据时，则可以暂时不执行该语句。Mybatis 默认是没有懒加载的，需要在 Myabis 的 mybatis.xml 配置文件中手动开启，同时在映射文件中设置 `fetchType` 启用懒加载。（连接查询是没有懒加载的）

```XML
<configuration>
    <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="cacheEnabled" value="true"/>
    </settings>
</configuration>
```

```xml
<resultMap id="userMap" type="user">
    <result column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="password" property="password"/>
    <result column="email" property="email"/>
    <!--property是我们1的成员变量名-->
    <!--colume是来源于第一次查询结果的列名，并且为第二次查询提供参数-->
    <association property="userDetail" fetchType="lazy" column="id" select="com.cskaoyan.mapper.UserDetailMapper.selectUserDetailByUid"/>
</resultMap>
```

### 缓存

为了减少磁盘的IO，mybatis 提供了缓存机制。

{{< figure src="/images/mybatis_cache.png" title="mybatis 缓存" >}}

#### 一级缓存

一级缓存默认是开启的，一级缓存级别是 sqlSession 级别的，即不同的 sqlSession 不会共享缓存。当 sqlSession commit 提交之后，缓存失效。同时若数据库的数据发生更改、添加和删除时，则缓存失效。

#### 二级缓存

 二级缓存默认是关闭的，二级缓存级别是 mapper 级别的，**当 sqlSession 提交之后，二级缓存才生成** ，同时不同的 sqlSession 对象（同一个类的不同 sqlSession 对象）的 mapper 也共享当前缓存，如果对数据库进行了增删改操作是，**如果没有提交事务，则缓存不会失效，即可能造成幻读**，只有提交事务之后缓存才失效，需要注意。

二级缓存需要在 mybatis.xml 配置文件中开启

```xml
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

如果要对某一个类的进行二级缓存查询时，必须实现序列化接口，进行序列化，否则无法缓存，同时在该类的映射文件在添加 `cache` 标签。

```java
public class User implements Serializable{
    ...
}
```

```xml
<cache/>
```

## 逆向工程

为了减少映射文件的书写，可以使用逆向工程生成映射文件。

### 导包

导包逆向工程的依赖和数据库的依赖

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.1</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.12</version>
    <scope>runtime</scope>
</dependency>
```

### 配置文件设置

在项目的根目录下配置逆向工程的配置 `generatorConfig.xml` 。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT"
                        userId="root"
                        password="admin">
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>
        <!--&lt;!&ndash;
            for oracle
           &ndash;&gt;
        <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
            connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:yycg"
            userId="yycg"
            password="yycg">
        </jdbcConnection>-->

        <!-- 默认false，
            为false把JDBC DECIMAL 和 NUMERIC 类型解析为Integer，
            为 true把JDBC DECIMAL 和 NUMERIC 类型解析为java.math.BigDecimal -->
        <!--<javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>-->

        <!-- javaModelGenerator javaBean生成的配置信息
             targetProject:生成PO类的位置
             targetPackage：生成PO类的类名-->
        <javaModelGenerator targetPackage="com.cskaoyan.bean"
                            targetProject="./mybatis_demo8/src/main/java">
            <!-- enableSubPackages:是否允许子包,是否让schema作为包的后缀
                 即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="true" />
            <!-- 从数据库返回的值是否清理前后的空格 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>


        <!-- sqlMapGenerator Mapper映射文件的配置信息
            targetProject:mapper映射文件生成的位置
            targetPackage:生成mapper映射文件放在哪个包下-->
        <sqlMapGenerator targetPackage="com.cskaoyan.mapper"
                         targetProject="./mybatis_demo8/src/main/resources">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--
           javaClientGenerator 生成 Model对象(JavaBean)和 mapper XML配置文件 对应的Dao代码
           targetProject:mapper接口生成的位置
           targetPackage:生成mapper接口放在哪个包下

           ANNOTATEDMAPPER
           XMLMAPPER
           MIXEDMAPPER
        -->

        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.cskaoyan.mapper"
                             targetProject="./mybatis_demo8/src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator><!---->
        <!-- 指定数据库表 -->

            <!-- 指定所有数据库表 -->

            <!--<table tableName="%"
                   enableCountByExample="false"
                   enableUpdateByExample="false"
                   enableDeleteByExample="false"
                   enableSelectByExample="false"
                   enableInsert="false"
                   enableDeleteByPrimaryKey="true"
                   enableSelectByPrimaryKey="true"
                   selectByExampleQueryId="false" ></table>-->

               <!-- 指定数据库表，要生成哪些表，就写哪些表，要和数据库中对应，不能写错！ -->
               <table  tableName="j13_user_t"
                       enableCountByExample="false"
                       enableUpdateByExample="false"
                       enableDeleteByExample="false"
                       enableSelectByExample="false"
                       enableInsert="true"
                       enableDeleteByPrimaryKey="true"
                       enableSelectByPrimaryKey="true"
                       selectByExampleQueryId="false"
                       domainObjectName="User"
               > </table>
                <table tableName="j13_student_t" domainObjectName="Student"/>
                <table tableName="j13_course_t" domainObjectName="Course"/>


        <!--      <table schema="" tableName="orders"></table>
             <table schema="" tableName="items"></table>
             <table schema="" tableName="orderdetail"></table>
      -->
               <!-- 有些表的字段需要指定java类型
                <table schema="" tableName="">
                   <columnOverride column="" javaType="" />
               </table> -->
    </context>
</generatorConfiguration>
```

### 引入逆向工程的 main 方法

配置 main 方法，并运行，即可以生成配置文件中设置好的映射配置文件。

```java
public class Generator {
    public void generator() throws Exception{
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true; //指向逆向工程配置文件
        File configFile = new File("mybatis/generatorConfig.xml");
        System.out.println(configFile.getAbsolutePath());
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator =
                new MyBatisGenerator(config, callback, warnings);

        myBatisGenerator.generate(null);
    }

    public static void main(String[] args) throws Exception {
        try {
            Generator generatorSqlmap = new Generator();
            generatorSqlmap.generator();
        }
        catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

## 其他

1. 若映射文件和映射类在同一个包下，且名字相同时，则在映射文件中 resultType 通常用类名小写来代表返回类型。也可以在类名上使用 `@Alias` 注解，来规定返回使用类型别名。

```java
@Alias("fruit")
public class Apple{
    ...
}
```

```xml
<select id="selectApple" resultType="fruit">
	select * from j13_apple_t
</select>
```

2. 对于简单的 sql 语句时，可以直接在映射接口上使用注解来完成

```java
public interface AppleManager{
    @Select("select * from j13_apple_t where id #{id}")
    Apple selectAppleById(@Param("id") int id);
}
```




