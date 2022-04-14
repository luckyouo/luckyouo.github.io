# log4j 日志配置


## 前言

为了记录代码的工作信息，入门操作通常是使用控制台输出来查看记录。而当代码需要长时间工作时，则需要持久化记录。

log4j 是 apache 下日志记录的常用软件，可以在终端输出和在文本内记录应用的工作记录。

## 使用流程

### 导包

导入 `log4j` 的依赖

- log4j 1.x 版本导入

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

- log4j 2.x 版本导入

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.2</version>
</dependency>
```

### 配置 log4j 的配置文件

- log4j 1.x 

```properties
### DEBUG?INFO?WARN?ERROR?FATAL
#????+??appender
log4j.rootLogger=debug ,file,stdout,file2

#Appender
#org.apache.log4j.ConsoleAppender?????
#org.apache.log4j.FileAppender????
#org.apache.log4j.DailyRollingFileAppender????????????
#org.apache.log4j.RollingFileAppender???????????????????????
#org.apache.log4j.WriterAppender?????????????????????
### direct log messages to stdout ###
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

### direct messages to file hibernate.log ###
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.File=/home/swq/Desktop/11mybatis.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

#org.apache.log4j.HTMLLayout??HTML???????
#org.apache.log4j.PatternLayout?????????????
#org.apache.log4j.SimpleLayout?????????????????
#org.apache.log4j.TTCCLayout????????????????????
log4j.appender.file2 = org.apache.log4j.FileAppender
log4j.appender.file2.File=/home/swq/Desktop/file2.log
log4j.appender.file2.layout=org.apache.log4j.SimpleLayout
#log4j.appender.file2.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n
```

- log4j 2.x 

2.x 已经放弃了 properties 方式，而是采用 xml, json 或者 jsn 格式。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- log4j2 配置文件 -->
<!-- 日志级别 trace<debug<info<warn<error<fatal -->
<configuration status="debug">
    <!-- 自定义属性 -->
    <Properties>
        <!-- 日志格式(控制台) -->
        <Property name="pattern1">[%-5p] %d %c - %m%n</Property>
        <!-- 日志格式(文件) -->
        <Property name="pattern2">
            =========================================%n 日志级别：%p%n 日志时间：%d%n 所属类名：%c%n 所属线程：%t%n 日志信息：%m%n
        </Property>
        <!-- 日志文件路径 -->
        <Property name="filePath">logs/myLog.log</Property>
    </Properties>
 
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${pattern1}"/>
        </Console>
        <RollingFile name="RollingFile" fileName="${filePath}"
                     filePattern="logs/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout pattern="${pattern2}"/>
            <SizeBasedTriggeringPolicy size="5 MB"/>
        </RollingFile>
    </appenders>
    <loggers>
        <root level="debug">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFile"/>
        </root>
    </loggers>
</configuration>
```

### 使用

- log4j 1.x 

```java
Logger logger = Logger.getLogger(LogTest.class);
logger.debug("debug");
logger.warn("warn");
logger.info("info");
logger.error("error");
logger.fatal("fatal");
```

- log4j 2.x 

```java
Logger logger = LogManager.getLogger(LogTest.class);
logger.trace("trace level");
logger.debug("debug level");
logger.info("info level");
logger.warn("warn level");
logger.error("error level");
logger.fatal("fatal level");
```

通过传入类名获得其 logger 对象，并使用其打印日志。

