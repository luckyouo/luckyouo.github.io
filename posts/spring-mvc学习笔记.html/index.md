# Spring MVC学习笔记


## 前言

MVC 三个单词分别为 `Model` 、 `View` 和 `Controller` ，MVC 模式是一种常用的web开发模式，比如python的 `django` 也是采用该模式。Spring 框架也提供了该模式，负责表示层。

{{< figure src="/images/spring_mvc.png" title="spring mvc流程图" >}}

## 使用流程

`Spring MVC` 提供了四大组件，**前端控制器（DispatcherServlet）、处理器映射器（HandlerMapping）、处理器适配器（HandlerAdapter）以及视图解析器（ViewResolver）**。

1. 用户请求发送到**前端控制器 DispatcherServlet**。
2. 前端控制器 DispatcherServlet 接收到请求后，DispatcherServlet 会使用 HandlerMapping 来处理，**HandlerMapping 会查找到具体进行处理请求的 Handler 对象**。
3. HandlerMapping 找到对应的 Handler 之后，并不是返回一个 Handler 原始对象，而是一个 Handler 执行链（HandlerExecutionChain），在这个执行链中包括了拦截器和处理请求的 Handler。HandlerMapping 返回一个执行链给 DispatcherServlet。
4. DispatcherServlet 接收到执行链之后，会**调用 Handler 适配器去执行 Handler**。
5. Handler 适配器执行完成 Handler（也就是 Controller）之后会得到一个 ModelAndView，并返回给 DispatcherServlet。
6. DispatcherServlet 接收到 HandlerAdapter 返回的 ModelAndView 之后，会根据其中的视图名调用 ViewResolver。
7. **ViewResolver 根据逻辑视图名解析成一个真正的 View 视图**，并返回给 DispatcherServlet。
8. DispatcherServlet 接收到视图之后，会根据上面的 ModelAndView 中的 model 来进行视图中数据的填充，也就是所谓的**视图渲染**。
9. 渲染完成之后，DispatcherServlet 就可以将结果返回给用户了。

## 使用流程

### 导包

导入 `spring-webmvc` 依赖，该依赖依赖于 `spring-context` ，所以可以不用手动导入 `spring-context`。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.17</version>
</dependency>
```

### 配置前端控制器

```xml
<servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:application.xml</param-value>
    </init-param>
</servlet>
    
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 配置 `HandlerMapping` 和 `HandlerAdapter` 

可以通过配置文件配置也可以通过注解配置。

1. 通过配置文件

在配置文件中配置 `springmvc` 的所需要的 `bean` 。

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>

<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>
```

2. 通过注解方式（常用）

在配置文件进行驱动注册（需要使用 spring-context 的注解扫描）

```xml
<context:component-scan base-package="com.cskaoyan"/>
<mvc:annotation-driven/>
```

### 配置Controller和Url

可以通过实现接口实现，也可以通过注解实现.

1. 实现接口实现。实现 `Controller` 接口，并重写 `handlerRequest` 方法。并在配置文件配置 `url` 映射。

```java
public class HelloController implements Controller{
    @Overriders
    public ModelView handleRequest(HttpServletRequest request, HttpServeletResponce responce){
        
    }
}
```

```xml
<bean name="hello" class="com.cskaoyan.controller.HelloController"></bean>
```

2. 通过注解实现。（常用）

```java
@Controller
public class HelloController{
    @RequestMapping("/hello")
    public ModelAndView login(){
        
    }
}
```

可以通过 `application.xml` 配置文件设置 `url` 路径的前缀和后缀，减少路径名书写。

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/view/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

### 拦截器配置（可选）

SpringMVC 的处理器拦截器类似于Servlet 开发中的过滤器Filter，用于对处理器进行预处理和后处理。

1. 实现 `HandlerIntercepter` 接口，并重写 `preHandler` 、 `postHandler` 和 `afterCompletion` 方法。
2. 在 `application.xml` 文件中配置拦截器。其中 `mapping` 表示配置的内容对应拦截的 `url` 范围。

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/user/*"/>
        <bean class="com.cskaoyan.hander.Myhandeler"/>
    </mvc:interceptor>
</mvc:interceptors>
```

- `preHandler` ：预处理回调方法，实现处理器的预处理（如登录检查），第三个参数为响应的处理器。返回值为 `boolean` ，true表示继续流程（如调用下一个拦截器或处理器），false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应
- `postHandler` ： 后处理回调方法，实现处理器的后处理（但在渲染视图之前），此时我们可以通过modelAndView（模型和视图对象）对模型数据进行处理或对视图进行处理，modelAndView也可能为null。
- `afterCompletion` ： 整个请求处理完毕回调方法，即在视图渲染完毕时回调，如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中preHandle返回true的拦截器的afterCompletion

可以配置多个拦截器，拦截范围相同时，则按注册顺序执行。只要 `preHandler` 返回为 `true` 时，`afterCompletion` 一定会执行到。

## 其他设置

### RequestMapping功能

1. url 路径映射
2. 窄化请求。可以在 `Controller` 类上同时配置 `RequestMapping` ，类里面的所有方法都在父路径里面。
3. 请求方法限定。使用 `Request.Method` 来限制请求方法。同时也可以通过其他参数配置限定其他参数。

### Controller 方法的返回值

返回值类型可以为如下三种：

1. `ModelAndView` : 返回模型视图对象，可以使用该对象来设置请求的视图。
2. `Void` ： 请求参数中必须使用 `(HttpServletRequest request, HttpServeletResponce responce)` ，使用 `request` 来设置请求视图。
3. `String` ： 返回视图对应的字符串。会自动转发到对应的视图文件上，也可也手动使用 `forward` 或者 `redirect` 转发到对应的视图的解析器上，也可以转发到其他 `Controller` 的  url 路径上。

```java
return "forward:hello.jsp";

return "redirect:hello.jsp";

return "redirct:/user/login";
```

### 请求参数封装

1. 使用 `request` 对象提取

在请求参数中，增加 `(HttpServletRequest request, HttpServeletResponce responce)` 参数，利用 `request` 对象根据 `key` 获取对应的 `value` 。

```java
String str = request.getParameter("id");
```

2. 基本数据类型获取

需要与请求参数 `key` 一一对应，即形参必须同名，通过相同的参数名提取。

3. 将请求参数封装到 `JavaBean` 对象当中（常用）

需要在视图页面使用 `对象.属性` 来设置传递过来的 `value` ，如果存在多个对象时，可以封装一个含有多个对象的对象。

```jsp
<input type="text" name="user.id">
```

如果在提交的过程中，使用了中文数据，则会导致，需要在配置文件 `web.xml` 中配置传输数据的编码格式。

```xml
<filter>
    <filter-name>ebcoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
    
<filter-mapping>
    <filter-name>ebcoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 请求参数类型转换

当请求的参数需要进行转换才能使用时，比如时间格式，需要对传入的数据进行格式转换。

1. 首先配置自定义的类型转换器。通过实现 `Converter` 接口，重写 `convert` 方法来实现类型转换器。

```java
public class MyDateCOnverter implements Converter<String, Date> {
    @Override
    public Date convert(String value) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd");
        try {
            Date parse = simpleDateFormat.parse(value);
            return parse;
        } catch (ParseException e) {
            e.printStackTrace();
        }

        return null;

    }
}
```

2. 在 `application.xml` 文件中，配置类型转换器。

```xml
<mvc:annotation-driven conversion-service="myConvertorService"/>

<bean id="myConvertorService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.cskaoyan.converter.MyDateCOnverter"/>
            <bean class="com.cskaoyan.converter.UserConverter"/>
        </set>
    </property>
</bean>
```

### 请求参数集合封装

1. 数组封装

使用 `checkbox`，传递多个值

```jsp
<h2>封装数组获取参数</h2>
<form action="submitArray" method="post">
    id1:<input  type="checkbox" name="ids" value="1"><br>
    id2:<input  type="checkbox" name="ids" value="2"><br>
    id3:<input  type="checkbox" name="ids" value="3"><br>
    id4:<input  type="checkbox" name="ids" value="4"><br>
    id5:<input  type="checkbox" name="ids" value="5"><br>
    <input type="submit">
</form>
```

2. 集合封装

使用索引设置 `ArrayList` 各个元素属性。同时构建一个 `JavaBean` 对象来接受请求参数，该对象含有对应的 `List<T>` 成员属性。

```java
public class QueryVo {

    User user;
    String content;
    int count;
    List<User> users;

    @Override
    public String toString() {
        return "QueryVo{" +
                "user=" + user +
                ", content='" + content + '\'' +
                ", count=" + count +
                ", users=" + users +
                '}';
    }

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }

    public QueryVo(User user, String content, int count) {
        this.user = user;
        this.content = content;
        this.count = count;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }
}
```

```jsp
<h2>封装集合获取参数</h2>
<form action="submitCollection" method="post">
    id:<input  type="text" name="user.id"><br>
    username:<input  type="text" name="user.username"><br>
    password:<input  type="password" name="user.password"><br>
    phoneNumber:<input  type="text" name="user.phoneNumber"><br>

    content:<input  type="text" name="content"><br>
    count:<input  type="text" name="count"><br>

    user1:id:<input  type="text" name="users[0].id"><br>
    user1:username:<input  type="text" name="users[0].username"><br>
    user1:password:<input  type="password" name="users[0].password"><br>
    user1:phoneNumber:<input  type="text" name="users[0].phoneNumber"><br>

    user2:id:<input  type="text" name="users[1].id"><br>
    user2:username:<input  type="text" name="users[1].username"><br>
    user2:password:<input  type="password" name="users[1].password"><br>
    user2:phoneNumber:<input  type="text" name="users[1].phoneNumber"><br>

    user3:id:<input  type="text" name="users[2].id"><br>
    user3:username:<input  type="text" name="users[2].username"><br>
    user3:password:<input  type="password" name="users[2].password"><br>
    user3:phoneNumber:<input  type="text" name="users[2].phoneNumber"><br>

    <input type="submit">
</form>
```

### 文件上传

图片上传需要外部依赖。

1. 导包。导入 `commons-io` 和 `commons-fileupload` 依赖。

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
```

2. 配置解析器。在 `application.xml` 配置解析器。（id 必须固定，否则异常）

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
</bean>
```

3. 接受参数中，增加 `MultipartFile myfile` 参数，使用 `transferTo` 方法传输的文件写入本地。

```java
String filePath = "path/to/file";
File file = new File(filePath);
myfile.transferTo(file);
```



## 参考资料

[[深入源码分析SpringMVC执行过程](https://segmentfault.com/a/1190000021848063)]

