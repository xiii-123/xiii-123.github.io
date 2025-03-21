---
title: 新版IDEA中Lombok不生效
published: 2025-03-18
description: 'Junit常用断言和注解'
image: ''
tags: [Lombok]
category: 'java-web'
draft: false 
---
今天写Springboot项目时，写了一个简单的`User`类以及一个`UserController`，但是编译的时候报错：找不到`User`类的全参构造方法。然后又试了`get`方法也报错找不到。于是猜测是`Lombok`未生效。

去网上找了一下，发现是新版IDEA的锅，需要将自动导入pom.xml文件中的一个plugin注释掉：
```xml
    <build>
        <plugins>
<!--            <plugin>-->
<!--                <groupId>org.apache.maven.plugins</groupId>-->
<!--                <artifactId>maven-compiler-plugin</artifactId>-->
<!--                <configuration>-->
<!--                    <annotationProcessorPaths>-->
<!--                        <path>-->
<!--                            <groupId>org.projectlombok</groupId>-->
<!--                            <artifactId>lombok</artifactId>-->
<!--                        </path>-->
<!--                    </annotationProcessorPaths>-->
<!--                </configuration>-->
<!--            </plugin>-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
注释之后刷新`maven`，项目启动成功。