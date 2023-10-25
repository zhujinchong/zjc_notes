# SLF4J+logback

## 日志简介

Java标准库提供了 java.util.logging 来实现日志功能，但是管理不方便。

Commons Logging是一个第三方日志库，它是由Apache创建的日志模块。它的特色是，它可以挂接不同的日志系统，并通过配置文件指定挂接的日志系统。默 认情况下，Commons Loggin自动搜索并使用Log4j（Log4j是另一个流行的日志系统），如果没有找 到Log4j，再使用JDK Logging。

Commons Logging和Log4j这一对好基友，它们一个负责充当日志API，一个负责实现日 志底层，搭配使用非常便于开发。

因为他们不开源，所以有人实现了SLF4J接口，有人实现了logback日志系统。

并且SpringBoot2自带，且默认使用他们。

## 快速入门

在resources中新建文件logback-spring.xml，此名称springboot会自动识别加载。

logback-spring.xml内容如下：

```
<?xml version='1.0' encoding='UTF-8'?>
<!--日志配置 参考 https://zhuanlan.zhihu.com/p/339741978 -->
<configuration>
    <!--直接定义属性-->
    <!--log文件位置-->
    <property name="logFile" value="logs/mutest"/>
    <property name="maxFileSize" value="30MB"/>

    <!--控制台日志-->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d [%thread] %-5level %logger{50} -[%file:%line]- %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--滚动文件日志：先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
    <appender name="fileLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logFile}.log</file>
        <encoder>
            <!--日志输出格式-->
            <pattern>%d [%thread] %-5level -[%file:%line]- %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!--每天生成一个新的活动日志文件，旧的日志归档，后缀名为2019.08.12这种格式-->
            <fileNamePattern>${logFile}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!--活动日志文件最大值，超过这个值将产生新日志文件-->
            <maxFileSize>${maxFileSize}</maxFileSize>
            <!--只保留最近3天的日志-->
            <maxHistory>3</maxHistory>
            <!--用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志-->
            <totalSizeCap>200MB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!--日志输出级别，默认是debug-->
    <root level="info">
    </root>
</configuration>
```

测试：

```
@GetMapping("/log")
@ResponseBody
public String logTest() {
    String name = "tom";
    int age = 18;
    logger.info("logTest, name:{}, age{}", name, age);
    return "hello";
}
```

## 配置文件详解





