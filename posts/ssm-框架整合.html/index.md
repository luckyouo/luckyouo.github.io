# SSM 框架整合


## 前言

`SSM` 框架缩写是由 `Spring ` 、 `Spring MVC` 和 `Mybatis` 三个单词首字母组合而成，将三者组合一起使用，就是常说的 `SSM` 框架。

## 通过 xml 文件整合 ssm

### 导包

导入三个组件的依赖，其中 mybatis 提供了整合 spring 的 jar 包（mybatis-spring）

```xml
<dependencies>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.17</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.12</version>
        <scope>runtime</scope>
    </dependency> 
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.6</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.3.17</version>
    </dependency>
    <dependency>
        <groupId>com.mchange</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.5.2</version>
    </dependency>
</dependencies>
```

### 配置 sqlSessionFactory

在 application.xml 配置文件中，向容器中注册连接池 datasource，配置 c3p0 的datasource 连接池（后续更换 druid 连接池，性能更换好）。在向 spring 容器中注册 sqlSessionFactoryBean，后续通过其获得 sqlSession对象。

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="typeAliasesPackage" value="com.cskaoyan.bean"/>
    <property name="dataSource" ref="datasource"/>
</bean>

<bean id="datasource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT"/>
    <property name="user" value="root"/>
    <property name="password" value="admin"/>
</bean>
```

### 获取 mapper 对象

1. 通过 mapper 接口的实现类。

```java
@Repository
public class UserMapperImpl implements UserMapper {
    @Autowired
    SqlSessionFactory sqlSessionFactory;

    @Override
    public User queryUserById(int id) {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        return null;
    }
}
```

2. 通过配置文件

在 application.xml 向容器中注册 MapperFactoryBean 对象。

```xml
<bean class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="mapperInterface" value="com.cskaoyan.mapper.UserMapper"/>
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>
```

该方法缺点是每个 mapper 接口都需要在配置文件中编写 MapperFactoryBean。

3. 注册 MapperScannerConfigurer 对象（常用方法，后续换成注解方式）

向容器中注册映射接口扫描对象，并配置扫描范围，使用 sqlSessionFactory 来生成映射对象。

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.cskaoyan.mapper"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

### 配置 Spring MVC

在 web.xml 文件中配置 DispatherServlet  

```xml
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:application.xml</param-value>
    </init-param>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

在 application.xml 注册 mvc 扫描驱动和 事务驱动

```xml
<context:component-scan base-package="com.cskaoyan"/>
<mvc:annotation-driven/>

<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="datasource"/>
</bean>
<tx:annotation-driven transaction-manager="txManger"/>
```

### 父子容器配置

上述过程将 spring 和 spring mvc 配置在一块，可以将二者分来开来，其中 spring 作为主容器，spring mvc 作为子容器。

在 web.xml 文件中使用 listener 标签加载 spring 容器，并设置 mvc 子容器的配置文件所在位置。

```xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:application.xml</param-value>
</context-param>


<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:application_mvc.xml</param-value>
    </init-param>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

父容器的配置文件为 applicatio.xml，子容器的配置为 application.xml。

父容器的配置内容为：其中 SpringBean 为 spring 容器的配置类，用来向 Spring 容器中注册类。

```xml
<context:component-scan base-package="com.cskaoyan">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>


<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="typeAliasesPackage" value="com.cskaoyan.bean"/>
    <property name="dataSource" ref="datasource"/>
</bean>

<bean id="datasource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT"/>
    <property name="user" value="root"/>
    <property name="password" value="admin"/>
</bean>

<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.cskaoyan.mapper"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>

<bean class="com.cskaoyan.bean.SpringBean"/>
```

子容器配置内容为：其中 SpringMvcBean 容器的配置类，用来向 Spring MVC 容器中注册类。

```xml
<context:component-scan base-package="com.cskaoyan.controller"/>
<mvc:annotation-driven/>
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/view/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<bean class="com.cskaoyan.bean.SpringMvcBean"/>
```

## 通过配置类整合 ssm 

上面通过配置 xml 文件来实现 ssm 整合，也可以通过配置类来实现 ssm 整合。

### 配置启动器类

新建一个启动类，继承抽象类 `AbstractAnnotationConfigDispatcherServletInitializer` ，并重写父类方法的三个方法。`getRootConfigClasses` 方法是获得 Spring 容器配置类对象，`getServletConfigClasses` 方法获得 Spring MVC 容器配置类对象，`getServletMappings` 设置 `DispatcherServlet` 的启动路径。

```java
public class ApplicationInitialize extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};

    }
}
```

### 配置 Spring 配置类

配置类上使用 `@Configuration` 和 `@ComponentScan` 注解，第一个注解是表示该类为一个配置类，第二个配置扫描注解的范围，若在扫描范围内，且含有对应注解，则向 Spring 容器中注册，要注意排除 `Spring MVC 注册类扫描的范围` ，否则会向容器生成两个对象。并向 Spring 容器中注册 `SqlSessionFactoryBean` 、`ComboPooledDataSource` 和 `MapperScannerConfigurer` 对象。

```java
@Configuration
@ComponentScan(basePackages = "com.cskaoyan", excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class, EnableWebMvc.class})})
public class SpringConfig {


    @Bean("sqlSessionFactoryBean")
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setTypeAliasesPackage("com.cskaoyan.bean");
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }

    @Bean
    public DataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
        comboPooledDataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
        comboPooledDataSource.setJdbcUrl("jdbc:mysql://localhost:3306/j13_jdbc_template?serverTimezone=GMT");
        comboPooledDataSource.setUser("root");
        comboPooledDataSource.setPassword("admin");
        return comboPooledDataSource;
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactoryBean");
        mapperScannerConfigurer.setBasePackage("com.cskaoyan.mapper");
        return mapperScannerConfigurer;

    }
}
```

### 配置 Spring MVC 配置类

在配置类上使用 `EnableWebMvc` 和 `ComponentScan` 注解，并实现 `WebMvcConfigurer` 接口。第一个注解表示该类为 Spring MVC 配置类，第二个配置为扫描注解范围，在 Spring MVC 配置类上配置视图解析器。

```java
@EnableWebMvc
@ComponentScan(basePackages = "com.cskaoyan.controller")
public class SpringMvcConfig implements WebMvcConfigurer {

    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
        internalResourceViewResolver.setSuffix(".jsp");
        internalResourceViewResolver.setPrefix("/WEB-INF/view/");
        return internalResourceViewResolver;
    }

}
```

### 其他组件

1. **Encoding Filter**

为了防止中文传输乱码，在启动类配置文件编码格式

```java
@Override
protected Filter[] getServletFilers(){
    CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
    encodingFilter.setEncoding("utf-8");
    encodingFilter.setForceEncoding(true);
    return new Filter[]{encodingFilter};
}
```

2. **Converter**

数据转换器，在 Spring MVC 配置类中注册转换器

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.cskaoyan.controller")
public class SpringMvcConfig implements WebMvcConfigurer {
    @Autowired
    ConfigurableConversionService configurableConversionService;
    
    @Bean
    @Primary
    public ConfigurableConversionService configurableConversionService(){
        return configurableConversionService;
    }
    
    @PostConstruct
    public void addConverterToService(){
        configurableConversionService.addConverter(new MyConverter());
    }
}
```

3. **File upload**

可以通过注册 bean 进行文件上传

```java
@Bean
public CommonMultipartResolver multipartResolver(){
    CommonMultipartResolver commonMultipartResolver = new CommonMultipartResolver();
    commonMultipartResolver.setMaxUploadSize(500000);
	return commonMultipartResolver;
}
```


