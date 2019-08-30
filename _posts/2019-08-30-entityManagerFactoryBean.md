---
layout: post
title: LocalContainerEntityManagerFactoryBean 에 null 를 리턴할 때
description: Spring Configuration 에서 직접 EntityManagerFactory 를 등록할 때 null 리턴할 때 잊은 것
date:   2019-08-30 13:00:00 +0900
author: Jeongjin Kim
categories: Spring
tags:	Spring JPA EntityManagerFactory
---

`LocalContainerEntityManagerFactoryBean` 을 이용해서 직접 Bean 을 등록할 때

```java
LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
return emf.getObject();
```

를 하면 null 이 리턴된다.

이유는 `LocalContainerEntityManagerFactoryBean` 는 `AbstractEntityManagerFactoryBean` 을 상속하여 구현하는데
이 클래스의 메소드 `afterPropertiesSet()` 가 모든 속성값이 세팅이 다 됐음을 알리는 역할을 한다.

XML 로 Bean 설정을 하면 `BeanFactory` 가 이 메소드를 호출하지만 `Annotation` 으로 Bean 을 등록할 때에는 직접 호출해줘야 한다.

따라서 위 코드에서 `getObject()` 를 호출하기 전에 `afterPropertiesSet()` 을 호출하여 설정이 완료됐음을 알리고 오브젝트를 가져와야 한다.

```java
LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
emf.afterPropertiesSet()
return emf.getObject();
```