---
title: LogBack使用
published: 2025-03-22
description: 'LogBack使用'
image: ''
tags: [LogBack]
category: 'java-web'
draft: false 
---
Logback 是一个开源的日志框架，它是 Log4j 的后续产品，由同一个开发者开发。Logback 主要由三个模块组成：logback-core、logback-classic 和 logback-access。其中，logback-classic 是最常用的模块，它实现了 SLF4J（Simple Logging Facade for Java）接口，可以作为 Log4j 的替代品。

Logback 的配置文件主要用于定义日志记录的策略，包括日志级别、输出目标、格式等。配置文件可以是 XML、Groovy 或 properties 格式，其中 XML 格式最为常见。
## 使用：
1. 在类中添加变量`Logger`
   ```java
   private static final Logger logger = LoggerFactory.getLogger(YourClass.class);
   ```
2. 使用`SlF4J`注解
   ```java
   @Slf4j
   public class YourClass {
       // ...
   }
   ```

## 配置文件：
```xml
<configuration>
    <!-- 定义变量 -->
    <property name="LOG_DIR" value="/var/logs/myapp" />
    <!-- 定义 appender，用于指定日志输出目标 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 定义输出格式 -->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 指定日志文件路径 -->
        <file>${LOG_DIR}/myapp.log</file>
        <!-- 定义滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 指定滚动文件名模式 -->
            <fileNamePattern>${LOG_DIR}/archived/myapp.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 保持历史记录的最大天数 -->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <!-- 定义输出格式 -->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <!-- 定义日志记录器 -->
    <root level="DEBUG">
        <!-- 指定使用哪个 appender -->
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
    <!-- 为特定包或类定义日志级别 -->
    <logger name="com.mycompany.myapp" level="INFO"/>
</configuration>
```
这个配置文件中包含了以下几个主要部分：
1. **变量定义**：使用 `<property>` 标签定义变量，可以在配置文件的其他地方引用。
2. **Appender 定义**：使用 `<appender>` 标签定义日志输出目标。可以有多个 appender，例如控制台输出（ConsoleAppender）、文件输出（FileAppender）、滚动文件输出（RollingFileAppender）等。
3. **输出格式定义**：在每个 appender 内部，使用 `<encoder>` 标签定义日志的输出格式。
4. **滚动策略定义**：对于文件输出，可以定义滚动策略，例如按时间滚动（TimeBasedRollingPolicy）或按文件大小滚动（SizeBasedTriggeringPolicy）。
5. **日志记录器定义**：使用 `<root>` 标签定义根日志记录器，可以指定日志级别和使用的 appender。还可以使用 `<logger>` 标签为特定的包或类定义日志级别。
6. **日志级别**：日志级别包括 TRACE、DEBUG、INFO、WARN、ERROR，用于控制日志的详细程度。
这个配置文件是一个基本的示例，Logback 还支持更多高级功能和配置选项，可以根据具体需求进行定制。
