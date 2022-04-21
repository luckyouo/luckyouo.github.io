# Spring JDBC 学习笔记


## 前言

Spring 提供了 `JDBCTemplate` 来操作jdbc，而让用户只需要关注sql语句即可。

## 使用流程

### 使用JDBCTemplate

1. 导包。jdbc 所需要而依赖不在 spring 四个核心当中，需要额外导包。需要导入一个数据库驱动、一个连接池取代、 `spring-jdbc` 。数据库以 `mysql` 为例

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.12</version>
</dependency>
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.5</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.17</version>
</dependency>
```

2. 创建连接池

```java
ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
comboPooledDataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
comboPooledDataSource.setJdbcUrl("jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT");
comboPooledDataSource.setUser("root");
comboPooledDataSource.setPassword("admin");
```

3. 创建 jdbc 模板

```java
JdbcTemplate jdbcTemplate = new JdbcTemplate();
jdbcTemplate.setDataSource(comboPooledDataSource);
```

4. 模板使用

```java
String s = jdbcTemplate.queryForObject(sql, String.class, 0);
```

### 使用JDBCDaoSupport

方法与上述不同

1. 首先每个DAO都需要声明template 而且要写set方法
2. 在配置文件中配置JDBC连接池

```XML
<bean id="myDatasource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT"/>
    <property name="user" value="root"/>
    <property name="password" value="admin"/>
</bean>

<bean>
	<property name="jdbcTemplate" ref="myJDBCTemplate"></property>
    <property name="datasource" ref="myDatasource"></property>
</bean>
```

3. JDBCDaoSupport 中含有 `JdbcTemplate` 对象，直接使用即可


