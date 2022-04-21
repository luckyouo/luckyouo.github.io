# Freemarker 学习笔记


## 前言

在 `Spring MVC` 中，视图默认使用 `jsp` 文件，导致每次加载视图时，都需要对文件进行编译，从而浪费服务器的资源。为了减少对服务器资源使用，可以使用静态文件，提前将视图渲染为 `html` 文件。

`FreeMarker` 是一种是一款 *模板引擎*： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件

{{< figure src="/images/freemarker.png" title="freemarker 工作流程图" >}}

## 使用流程

1. 导入依赖。导入 `spring-context-support` 和 `freemarker` 依赖

```xml
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.3.17</version>
</dependency>
```

2. 在 `appliction.xml` 文件中配置对应的 `bean`对象。

```xml
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker"/>
    <property name="defaultEncoding" value="utf-8"/>
</bean>

<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="suffix" value=".ftl"/>
    <property name="contentType" value="text/html;ch  araset=utf-8"/>
    <property name="order"  value="1"/>
</bean>
```

3. 在需要渲染的视图中中，配置模板对象。

```java
public class MainTest{
    @Test
    public void myTest() throws IOException, TemplateException {
        // 使用freemark版本号，创建 Configuration 对象
        Configuration configuration = new Configuration(Configuration.getVersion());

        
        File file = new File("src/main/resources");
		// 设置模板对象所在路径		 
        configuration.setDirectoryForTemplateLoading(file);
        // 设置模板编码格式
        configuration.setDefaultEncoding("utf-8");
		// 创建模板对象
        Template template = configuration.getTemplate("hello.ftl");
		// 创建模板使用的数据集合，可以是 pojo 也可以是哈希表
        HashMap<String, Object> stringObjectHashMap = new HashMap<>();
        User user = new User("java", "pass", 10);
        stringObjectHashMap.put("user", user);
        stringObjectHashMap.put("code", "freemarker");
        // 创建一个 Writer对象，一般创建 FileWriter 对象，生成指定文件
        FileWriter fileWriter = new FileWriter(new File("src/main/resources/hello.html"));
		// 调用 process 方法，输出指定文件
   		template.process(stringObjectHashMap, fileWriter);
		// 关闭流
        fileWriter.close();
    }
}
```

### 基本语法

1. 访问哈希表中 `key` 所对应的 `value` 时，同 `jsp` 一样，使用 `${key}` 访问。 
2. 访问 `pojo` 中的属性：使用 `${类名.属性}` 访问
3. 访问集合中的数据： 使用 `#list` 标签遍历访问

```java
User user1 = new User("admin1", "123");
User user2 = new User("admin2", "123");
User user3 = new User("admin3", "123");

List<User> list = new ArrayList<>();
list.add(user1);
list.add(user2);
list.add(user3);
dataMap.put("userlist", list);
```

```ftl
<#list userlist as users>
	${users.username}
    ${users.password}
</#list>
```

4. 判断语法和比较语法

```ftl
<#if hello=="java">
    ${hello}
   <#elseif hello=="python"> 
    ${hello}
</#if>
```

5. 日期类处理

```ftl
${date?string("yyyy/MM/dd HH:mm:ss")}
```

6. null值处理：使用 `??` 判断是否为空，使用 `！` 表示为空使用默认值，后面什么都不跟的化，则使用空字符串。**（若为空使用的则会抛出异常，且会输出到默认的指定文件当中）**

```ftl
<#if hello??> 

${hello! "默认值"}
```

7. 引入其他模板文件。使用 `include` 引入其他模板文件。

```ftl
<#inclde "foo.ftl">
```


