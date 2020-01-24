---
layout: post
title:  "openapi日志工具logback配置说明"
categories: "ELK"
tags: "日志 logback"
author: "songzhx"
date:   2017-7-13
---

# openapi日志工具logback配置说明



>logback，类似log4j日志工具
>



### 步骤1：添加依赖包 

```xml
<dependency>  
      <groupId>ch.qos.logback</groupId>  
      <artifactId>logback-classic</artifactId>  
      <version>1.1.7</version>  
</dependency>
```



### 步骤2：添加指定的logback配置文件

新增加配置文件logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 日志文件根路径 -->
    <!-- 需要根据自己的日志 修改对应的文件路径 -->
    <property name="LOG_HOME" value="./logs/wzfw" />

    <!-- BUSSINESS控制台输出appender配置 -->
    <appender name="BUSSINESS_STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %msg%n
            </pattern>
        </encoder>
    </appender>
    <!-- Plain控制台输出appender配置 -->
    <appender name="PLAIN_STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- BUSSINESS类型日志输出appender配置
    ch.qos.logback.core.rolling.RollingFileAppender 为常用appender
    当日志大小超过策略后，将文件内容转储到指定文件
    -->
    <appender name="BUSSINESS_FILE_OUTPUT"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 业务类型日志工作文件路径 需要根据自己需求进行指定  -->
        <file>${LOG_HOME}/stat_logs/openapi-bussiness.log</file>
        <!-- 转储策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 转储文件路径，路径后加.gz或.zip，在转储后压缩 -->
            <fileNamePattern>${LOG_HOME}/stat_logs/openapi-bussiness.%d{yyyy-MM-dd}-%i.log
            </fileNamePattern>
            <!-- 最多保留文件数，系统保留最新的文件，超过该数量的删除 -->
            <maxHistory>20</maxHistory>
            <!-- 超过100MB或跨天后转储 -->
            <TimeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>100MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <!-- 日志格式 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %msg%n</pattern>
        </encoder>
    </appender>

    <!-- PLAIN类型日志输出appender配置
        ch.qos.logback.core.rolling.RollingFileAppender 为常用appender
        当日志大小超过策略后，将文件内容转储到指定文件
    -->
    <appender name="PLAIN_FILE_OUTPUT"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 工作文件路径 需要根据自己需求进行设置  -->
        <file>${LOG_HOME}/plain_logs/openapi-plain.log</file>
        <!-- 转储策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 转储文件路径，路径后加.gz或.zip，在转储后压缩 -->
            <fileNamePattern>${LOG_HOME}/plain_logs/openapi-plain-%d{yyyy-MM-dd}-%i.log
            </fileNamePattern>
            <!-- 最多保留文件数，系统保留最新的文件，超过该数量的删除 -->
            <maxHistory>10</maxHistory>
            <!-- 超过100MB或跨天后转储 -->
            <TimeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>1000MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <!-- 日志格式 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{0} - %msg%n</pattern>
        </encoder>
    </appender>

    <!--日志根目录-->
    <root level="INFO">
        <appender-ref ref="BUSSINESS_STDOUT" />
        <appender-ref ref="PLAIN_STDOUT" />
    </root>
    <!--针对业务类型日志设置-->
    <logger name="com.chinatelecom.bussiness" level="INFO" additivity="false">
        <appender-ref ref="BUSSINESS_FILE_OUTPUT" />
        <appender-ref ref="BUSSINESS_STDOUT" />
    </logger>
    <!--针对普通类型日志设置-->
    <logger name="com.chinatelecom.plain" level="INFO" additivity="false">
        <appender-ref ref="PLAIN_FILE_OUTPUT" />
        <appender-ref ref="PLAIN_STDOUT" />
    </logger>
</configuration>
```



###步骤3:代码引用logback工具

```java
public class LogbackTest {
    //业务类日志logger记录器
    private static final Logger bussiness_logger = LoggerFactory.getLogger("com.chinatelecom.bussiness");
    //普通类型日志logger记录器
    private static final Logger plain_logger = LoggerFactory.getLogger("com.chinatelecom.plain");
    @Test
    public void logbackTest(){
        bussiness_logger.info("this is  Info");
        plain_logger.info("this is plain info ");
        plain_logger.warn("this is warn");
        plain_logger.error("this is error");
        try {
            System.out.println(1 / 0);
        }catch (Exception e){
            plain_logger.error("App",e);
        }
    }
}
```







