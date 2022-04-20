# Shiro 学习笔记


## 前言

当用户访问网站时，需要判断用户用户是否登陆以及权限问题，这时需要对用户进行认证和授权。Spring 框架提供了 `spring security ` 权限框架，但是与 spring 过于耦合，而 shiro 是 apache 的一个开源框架，是一个权限管理的框架，实现 用户认证、用户授权.

{{< figure src="/images/shiro.png" title=" shiro 工作流程" >}}

## 使用流程

### 导入依赖

`shiro-spring` 提供了与 spring 整合的依赖，同时还附带了 shiro 的三个依赖。

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.25</version>
</dependency>
```

### SecurityManager 配置

SecurityManager 是 shiro 核心模块，需要对其进行初始化。

#### 使用配置文件初始化（了解）

当不使用配置类时，可以使用配置文件进行初始化。首先需要心配置 shiro 的配置文件 `shiro.ini` 。

```ini
[user]
#username=password
zhangsan=123456
```

使用 `IniSecurityManagerFactory` 和配置文件初始化工厂，并使用工厂对象获得 SecurityManager 的对象，并将 securityManager 设置为当前权限管理对象。

```java
Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```

#### 使用配置类时

```java
@Bean
public DefaultWebSecurityManager defaultWebSecurityManager(@Qualifier("userRealm") USerRealm userRealm, DefaultWebSessionManager defaultWebSessionManager){
    DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
    defaultWebSecurityManager.setRealm(userRealm);
    defaultWebSecurityManager.setSessionManager(defaultWebSessionManager);
    return defaultWebSecurityManager;
}
```

### 获取 Subject 对象

通过 SecurityUtils 即可以获取到 subject 对象，该对象调用 `login` 方法时，会执行 `Realm` 的认证和授权步骤。如果没有指定 `Realm` 时，则会执行默认的 `IniRealm` 下的方法，该默认的 `IniRealm` 方法会在 `ini` 文件查询用户信息。如果验证失败，则抛出异常。

```java
Subject subject = SecurityUtils.getSubject();
UsernamePasswordToken token = new UsernamePasswordToken(username, password);
try {
    subject.login(token);
    return "index";
}catch (UnknownAccountException e){
    model.addAttribute("msg", "用户名错误");
    return "login";
}catch (IncorrectCredentialsException e){
    model.addAttribute("msg", "密码错误");
    return "login";
}
```

### 自定义 Realm

默认的 Realm 无法满足用户需求，一般需要继承 `AuthorizingRealm` 对象，并重写认证和授权的方法。当返回结果为 null 时，则抛出异常。

```java
public class USerRealm extends AuthorizingRealm {
    @Autowired
    UserService userService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权");
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.addStringPermission("user:add");
        Set<String> set = new HashSet<>();
        set.add("user.add");
        return info;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证");
        UsernamePasswordToken usertoken = (UsernamePasswordToken) authenticationToken;
        User user = userService.queryUserByName(usertoken.getUsername());
        if(user == null){
            return null;
        }
        return new SimpleAuthenticationInfo("", user.getPassword(), "");
    }
}
```

#### 使用配置文件（了解）

需要在 shiro-ini 配置文件中设置 Realm

```ini
[main]
# 需要全类名
customReal=com.cskaoyan.shiro.CustomShiro 
securityManager.realm=$userRealm
```

#### 使用配置类

需要在 SecurityManager 对象内设置自定义的 Realm。

```java
defaultWebSecurityManager.setRealm(userRealm);
```

### 配置Filter

服务器内的资源并不是对所有用户都是可以访问的，需要对资源进行权限管理（也可以对用户进行权限管理，但用户管理起来更加不方便，同时是对用户进行角色划分，同时对资源根据不同的角色进行权限管理），这时需要使用过滤器对不同的 url 进行权限设置。

#### 使用配置文件

首先在 web.xml 文件中配置 shiro 的过滤器。

```xml
<filter>
    <filter-name>shiro</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>shiroFilter</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>shiro</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

接着 spring 的配置文件 application.xml 配置过滤选项，验证 url 路径由上至下依次匹配。

```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
    <property name="loginUrl" value="/login"/>
    <property name="unauthorizedUrl" value="/unAuthor"/>
    <property name="filterChainDefinitions">
        <value>
            /refuse = anon
            /index.jsp = anon
            /login.jsp = anon
            /login = anon
            /logout = logout
            /** = authc
        </value>
    </property>
</bean>
```

| 标签名称                            | 标签条件（均是显示标签内容） |
| ----------------------------------- | ---------------------------- |
| <shiro:authenticated>               | 登录之后                     |
| <shiro:notAuthenticated>            | 不在登录状态时               |
| <shiro:guest>                       | 用户在没有RememberMe时       |
| <shiro:user>                        | 用户在RememberMe时           |
| <shiro:hasAnyRoles name="abc,123" > | 在有abc或者123角色时         |
| <shiro:hasRole name="abc">          | 拥有角色abc                  |
| <shiro:lacksRole name="abc">        | 没有角色abc                  |
| <shiro:hasPermission name="abc">    | 拥有权限资源abc              |
| <shiro:lacksPermission name="abc">  | 没有abc权限资源              |
| <shiro:principal>                   | 显示用户身份名称             |

 <shiro:principal property="username"/>   显示用户身份中的属性值

当认证出现错误时，返回给用户的不应该是一个异常，而是需要对其进行重定向进行重新认证，上述配置中的 `unauthorizedUrl` 异常对应为认证异常重定向的 url ，`loginUrl` 对应的为登陆的 url ，logout 为注销的的url ，该 url 可以不配置映射即可以使用。

#### 使用配置类

通过 `ShiroFilterFactoryBean` 工厂对象设置 url 权限。该权限为一个 map，key 对应的是 url 路径，而 value 对应的为所需要的权限。

```java
filterMap.put("/user/add", "perms[user:add]");
filterMap.put("/user/*", "authc");
shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
shiroFilterFactoryBean.setLoginUrl("/tologin");
shiroFilterFactoryBean.setUnauthorizedUrl("/noauth");
```

当存在大量的 url 时，使用 map 管理起来可能不方便，则可以使用注解的方式，则需要在相应的 controller 配置权限控制。

使用注解是，需要导入 aop 技术，即需要导入 `aspectjweaver` 依赖，同时在 application-mvc.xml 中注册 advisor 对象。

```xml
<dependency>
	<groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.6.12</version>
</dependency>
```

```xml
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
    <property name="securityManager" ref="securityManager"/>
</bean>
```

```java
@RequestMapping("/user/query")
@RequiresPermissions(value = {"user:query", "user:delete"}, logical = Logical.OR)
public String queryUser(String name, Model model){
    User user = userService.queryUserByName(name);
    model.addAttribute("user", user);
    return "/WEB-INF/view/user.jsp";
}
```

Shiro共有5个注解， 详细如下

- **RequiresAuthentication:**

使用该注解标注的类，实例，方法在访问或调用时，当前Subject必须在当前session中已经过认证。

- **RequiresGuest:**

使用该注解标注的类，实例，方法在访问或调用时，当前Subject可以是“gust”身份，不需要经过认证或者在原先的session中存在记录。

- **RequiresPermissions:**

当前Subject需要拥有某些特定的权限时，才能执行被该注解标注的方法。如果当前Subject不具有这样的权限，则方法不会被执行。

- **RequiresRoles:**

当前Subject必须拥有所有指定的角色时，才能访问被该注解标注的方法。如果当天Subject不同时拥有所有指定角色，则方法不会执行还会抛出AuthorizationException异常。

- **RequiresUser**

当前Subject必须是应用的用户，才能访问或调用被该注解标注的类，实例，方法。

**注意：Shiro的认证注解处理是有内定的处理顺序的，如果有个多个注解的话，前面的通过了会继续检查后面的，若不通过则直接返回，处理顺序依次为（与实际声明顺序无关）**

## 其他

使用 shiro 对查询用户的权限时，都是通过访问数据库进行查询，为了减少磁盘的 IO，可以使用缓存来减少磁盘的 IO，shiro 默认是关闭认证信息缓存的，对于授权信息的缓存shiro默认开启的，但需要导入依赖和配置其 cacheManager。

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.2.3</version>
</dependency>
<!-- 缓存ehcache -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.8.1</version>
</dependency>
```

同时注册 EhCacheManager，并在 SecurityManager 中设置相应的 cacheManager。

```xml
<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
	<property name="cacheManagerConfigFile" value="classpath:shiro-ehcache.xml"/>
</bean>

<bean id="securtyManager" class="org.apache.shiro.web.mgt.Default.WebSecurityManager">
	<property name="cacheManager" ref="cacheManager"/>
</bean>
```

当用户权限时，缓存的信息不会更改，如果权限发生变化时，则需要更新缓存。可以在更新权限的同时，同时调用更新缓存的方法。


