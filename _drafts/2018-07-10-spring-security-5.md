---
layout: post
title: Cloud Native Java 따라하기 1
description: Cloud Native Java 클라우드 네이티브 자바를 따라해보자
date:   2018-07-16 13:00:00 +0900
author: Jeongjin Kim
categories: Cloud Spring Java
tags:	Cloud Spring Java
---

내용은 좋을지 모르겠지만 학습자 입장에서 예제를 따라하기가 힘들다. 완전한 소스를 제공해 주지 않고, 진행 흐름이 끊기고, 하던 프로젝트가 바뀐다.


## 의존성 설정
spring-boot initializr 를 활용하면 쉽게 추가할 수 있다. jpa, data-rest, web, h2, hateoas를 추가했다.
이외에 ide에 [lombok](https://projectlombok.org) plug-in을 설치하고 lombok 의존성을 추가해서 코드를 좀 더 간결하게 작성할 수 있도록 했다.

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
		   http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cothe</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-hateoas</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```

## Customer 엔티티 클래스 만들기

책에는 이 클래스를 만드는 부분이 없어 앞쪽에 나오는 Cat 클래스를 참고하여 유사하게 작성했다.

`Customer.java`

```java
package cothe.demo;

import lombok.Getter;
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
@Getter
public class Customer {
    @Id
    private long id;
    private String email;
    public Customer(long id, String email) {
        this.id = id;
        this.email = email;
    }

    public Customer() {
    }
}
```
## Spring 설정

`ApplicationConfiguration.java`

```java
package cothe.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;
import javax.sql.DataSource;

@Configuration
public class ApplicationConfiguration {
    @Bean
    DataSource dataSource() {
        return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2)
                .setName("customer").build();
    }

    @Bean
    CustomerService customerService(JdbcTemplate jdbcTemplate) {
        return new CustomerService(jdbcTemplate);
    }

    @Bean
    JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```
## CustomerReposity 만들기

`CustomerRepository.java`

```java
package cothe.demo;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource
public interface CustomerRepository extends JpaRepository<Customer, Long>{}
```

## CustomerService 만들기

`CustomerService.java`

```java
package cothe.demo;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import java.util.Collection;

public class CustomerService {
    private final JdbcTemplate jdbcTemplate;

    public CustomerService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Collection<Customer> findAll() {
        RowMapper<Customer> rowMapper =
                (rs, rowNum) ->
                        new Customer(rs.getLong("ID"), rs.getString("EMAIL"));

        return this.jdbcTemplate.query("select * from CUSTOMER", rowMapper);
    }
}
```

## Web Controller 만들기

`WebController.java`

```java
package cothe.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.Collection;

@RestController
public class WebController {
    @Autowired
    CustomerService customerService;

    @GetMapping(value = "/customers", produces = "application/hal+json")
    @ResponseBody
    Collection<Customer> customers(){
        return customerService.findAll();
    }
}
```

## 테스트 만들기
`DemoApplicationTests.java`

```java
package cothe.demo;

import java.util.Collection;
import java.util.HashMap;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.Assert.assertTrue;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class DemoApplicationTests {
    String[] sampleData = {"a@g.com", "b@k.com", "c@y.com"};
    @Autowired
    private MockMvc mvc; //REST 종단점(endpoint)에 요청을 보낼 수 있는 스프링 MVC 테스트의 MockMvc

    @Autowired
    private CustomerRepository customerRepository;

    @Before
    public void before() throws Exception {

        for (int i = 0; i < sampleData.length; i++) {
            customerRepository.save(new Customer(i, sampleData[i]));
        }
    }

    @Test
    public void customersReflectedInRead() throws Exception {
        MediaType halJson = MediaType.parseMediaType("application/hal+json;charset=UTF-8");
        this.mvc
                .perform(get("/customers"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(halJson))
                .andExpect(
                        mvcResult -> {
                            String contentAsString = mvcResult.getResponse().getContentAsString();
                            ObjectMapper objectMapper = new ObjectMapper();
                            Collection<Customer> customers =
                                    objectMapper.readValue(contentAsString,
                                            new TypeReference<Collection<Customer>>() {
                                            });

                            assertTrue(customers.size() == sampleData.length);
                        }
                );
    }
}
```


## AOP로 실행소요시간 로깅하기

`@EnableAspectJAutoProxy` 애노테이션을 추가하여 AspectAutoProxy기능을 활성화 시킨다.

```java
package cothe.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;
import javax.sql.DataSource;

@Configuration
@EnableAspectJAutoProxy
public class ApplicationConfiguration {
    @Bean
    DataSource dataSource() {
        return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2)
                .setName("customer").build();
    }

    @Bean
    CustomerService customerService(JdbcTemplate jdbcTemplate) {
        return new CustomerService(jdbcTemplate);
    }

    @Bean
    JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

## Aspect 클래스 작성

```java
package cothe.demo;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;
import java.time.LocalDateTime;

@Component
@Slf4j
@Aspect
public class LoggingAroundAspect {
    @Around("execution(* cothe.demo.CustomerService.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        LocalDateTime start = LocalDateTime.now();
        Throwable toThrow = null;
        Object returnValue = null;

        try {
            returnValue = joinPoint.proceed();
        } catch (Throwable throwable) {
            toThrow = throwable;
        }

        LocalDateTime stop = LocalDateTime.now();

        log.info("starting @ " + start.toString());
        log.info("finishing @ " + stop.toString() + " with duration " + stop.minusNanos(start.getNano()).getNano());


        if (null != toThrow) {
            throw toThrow;
        }
        return returnValue;
    }
}
```

## 클라우드 파운드리
배포 작업에 도움을 주는 클라우드 플랫폼이며 PaaS 형태로 사용한다. 구현되어 있는 것들 중에 [pivotal web service](https://run.pivotal.io) 를 이용한다.
계정을 만들고 [이곳](https://docs.run.pivotal.io/cf-cli/install-go-cli.html)을 참고하여 cf 명령행 인터페이스를 설치한다.

```sh
$ cf login

API 엔드포인트> https://api.run.pivotal.io

Email> xxxxxx@gmail.com
Password>
인증 중...
확인

대상 지정된 조직 cothe

대상 지정된 영역 development



API 엔드포인트:   https://api.run.pivotal.io(API 버전: 2.115.0)
사용자:           xxxxxx@gmail.com
조직:             cothe
영역:             development
```

설명이 완전하지 못 함. 작성 연기

---

# 12요소 애플리케이션 설정

스프링에서 설정(Configuration)이라고 말할 떄는 보통 빈을 어떻게 연결할지 컨테이너에게 알려주는 입력값이다.
설정은 XML로 작성되어 `ClassPathXmlApplicationContext` 통해 설정 될 수 있고, 자바 파일로 작성되어 `AnnotationConfigApplicationContext` 를 통해 설정 될 수 도 있다.

여기서 설정은 패스워드, 포트, 호스트 이름, 기능 활성화 여부 등 애플리케이션이 실행되는 환경에 따라 달라질 수 있는 갓을 의미한다.
이런 설정들은 소스 코드 상에 포함되면 안된다.