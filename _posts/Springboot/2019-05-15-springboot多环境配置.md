---
layout: post
title:  "springboot多环境配置和打包配置"
categories: "springboot"
tags: "springboot"
author: "songzhx"
date:   2019-05-15 14:46:00

---

## 多环境配置

**POM.xml配置**

```xml
<profiles>
		<profile>
			<!-- 本地开发环境 -->
			<id>dev</id>
			<properties>
				<profiles.active>dev</profiles.active>
			</properties>
			<!-- 默认是本地开发环境 -->
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
		</profile>
		<profile>
			<!-- 测试环境 -->
			<id>test</id>
			<properties>
				<profiles.active>test</profiles.active>
			</properties>
		</profile>
		<profile>
			<!-- 生产环境 -->
			<id>prod</id>
			<properties>
				<profiles.active>prod</profiles.active>
			</properties>
		</profile>
	</profiles>

```



**resource 文件配置**

```
application-dev.yml
application-prod.yml
application-test.yml
```



**执行语句**

```
打本地开发环境包：mvn clean package -Pdev
打部署上线环境包：mvn clean package -Ppro
打测试环境包：mvn clean package -Ptest
```



```java
如果使用命令行直接运行jar文件，则使用java -jar -Dspring.profiles.active=test demo-0.0.1-SNAPSHOT.jar

如果使用开发工具，运行Application.java文件启动，则增加参数--spring.profiles.active=test
```



## 打包配置

POM文件添加

```xml
	<build>
		<!-- maven模块化的话最好从父类继成取，打成包的命名 -->
		<finalName>${artifactId}-${version}-${profiles.active}</finalName>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<fork>
						true
					</fork>
				</configuration>
			</plugin>
		</plugins>
	</build>
```



## 其他配置



```
-Dlogging.config=/opt/esplan/logback-prod.xml
-Dlogging.path=/opt/esplan/logs
-Dserver.port=8080
```



具体执行：

```bash
java -Dlogging.config=/opt/esplan/logback-prod.xml -Dlogging.path=/opt/esplan/logs -Xmx4096m -Xms4096m -Dspring.profiles.active=prod -Dserver.port=8080 -jar esplan.jar 
```





## 参考：

<https://www.jianshu.com/p/5650e5738d30>

