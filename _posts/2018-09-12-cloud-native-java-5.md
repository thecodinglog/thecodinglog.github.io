---
layout: post
title: Cloud Native Java 따라하기 5(설정서버에 보안적용)
description: 스프링 클라우드 설정 서버 클라이언트 보안 적용 building spring clound configuation client server security
date:   2018-09-10 13:00:00 +0900
author: Jeongjin Kim
categories: Cloud Spring Java
tags:	Cloud Spring Java Spring-boot
---

**git 저장소**에 HTTP 기본 인증과 같은 보안 처리가 되어 있으면 **서버**에 인증 정보가 있어야 접근이 가능하다.
* spring.cloud.config.server.git.username 
* spring.cloud.config.server.git.passwaord

마찬가지로 서버 자체에 보안설정이 되어 있으면 클라이언트에 인증 정보가 있어야 접근이 가능하다. 서버 보안을 설정하는 가장 간단한 방법은 설정 서버에 스프링 시큐리티를 의존 관계로 추가하고, 클라이언트에서 인증 정보와 함께 요청하는 것이다.

* spring.security.user.name
* spring.security.user.password

## 스프링 클라우드 설정 서버에 보안 설정
### 스프링 시큐리티 의존성 추가

`pom.xml`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 접속 가능한 사용자 추가

`application.yml`

```yml
spring:
  security:
    user:
      name: user1
      password: pass
```

## 스프링 클라우드 설정 클라이언트 설정
### 접속 uri 바꾸기

`bootstrap.yml`

```yml
spring:
  cloud:
    config:
      uri: http://user1:pass@localhost:8889
```

이렇게 하면 간단하게 설정 서버에 보안 기능을 추가할 수 있다.