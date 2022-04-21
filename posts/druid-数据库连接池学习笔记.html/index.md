# Druid 数据库连接池学习笔记


## 前言

`Druid` 是 Alibab 开源的数据连接池，目前Java社区使用广泛的的数据库连接池，在功能、性能、扩展性方面，都超过其他数据库连接池，包括DBCP、C3P0、BoneCP、Proxool、JBoss DataSource。

## 使用流程

### 导入依赖

在 pom 文件导入 druid 相关依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>${druid-version}</version>
</dependency>
```

### 更改连接池

之前学习 Spring 框架时候用的是 c3p0 连接池，现在只需要更换连接池类就可以了。

```xml
<bean id="datasource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT"/>
    <property name="username" value="root"/>
    <property name="password" value="admin"/>
    <property name="filters" value="stat"/>
</bean>
```

### 开启性能监控

在 web.xml 文件中添加如下配置内容

```xml
<servlet>
    <servlet-name>DruidStatView</servlet-name>
    <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>DruidStatView</servlet-name>
    <url-pattern>/druid/*</url-pattern>
</servlet-mapping>
<!--  配置druid监控-->
<filter>
    <filter-name>DruidWebStatFilter</filter-name>
    <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
    <init-param>
        <param-name>exclusions</param-name>
        <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>DruidWebStatFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 查看

启动服务器，在 `http://localhost/yourproject/druid/index.html` 访问监控页面。

## 参考资料

[官方仓库 ](https://github.com/alibaba/druid)


