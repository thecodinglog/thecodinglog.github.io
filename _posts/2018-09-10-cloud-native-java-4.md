---
layout: post
title: Cloud Native Java 따라하기 4(스프링 클라우드 설정 클라이언트)
description: 스프링 클라우드 설정 클라이언트 만들기 building spring clound configuation client
date:   2018-09-10 13:00:00 +0900
author: Jeongjin Kim
categories: Cloud Spring Java
tags:	Cloud Spring Java Spring-boot
---

## 스프링 클라우드 설정 클라이언트

* 스프링 클라우드 기반 서비스는 실행할 때 `src/main/resources/bootstrap.properties`(또는 yml) 파일을 찾는다. 
* 애플리케이션 이름은 `spring.application.name` 속성에 지정한다. 
* bootstrap 파일은 application.properties 파일보다 먼저 로딩된다.
* 클라우드 설정 서버는 클라이언트의 `spring.application.name` 에 설정된 이름으로 설정파일을 찾는다. 여기서는 config-client 으로 설정했다.
* 설정서버에 저장된 application.properties 파일은 모든 설정 클라이언트에서 참조할 수 있다.

### 클라이언트 만들어 보기

먼저 새로운 maven 프로젝트를 만들어 의존성을 추가한다.

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cothe</groupId>
    <artifactId>springConfigClient</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-config</artifactId>
                <version>2.0.2.BUILD-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
    </dependencies><repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>

</project>
```

resources 디렉토리 밑에 bootstrap.yml 설정파일을 만들고 애플리케이션 이름과 설정서버 접속주소를 넣는다.

`bootstrap.yml`

```yml
spring:
  application:
    name: config-client
  cloud:
    config:
      uri: http://localhost:8888
```

빈에 설정된 정보를 매핑해서 실제로 값이 잘 넘어 오는지 로그를 확인한다.

`ConfigClientApplication.java`

```java
package cothe;

@SpringBootApplication
public class ConfigClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class);
    }

    @Bean
    public String test(@Value("${configuration.projectName}") String pn){
        System.out.println(">>> project name : "+pn);
        return pn;
    }
}
```

서버에 설정된 값이 클라이언트에 잘 넘어오는 것을 볼 수 있다.

![](/assets/2018-09-10-cloud-native-java-4/2018-09-10-cloud-native-java-4_143534.png)


