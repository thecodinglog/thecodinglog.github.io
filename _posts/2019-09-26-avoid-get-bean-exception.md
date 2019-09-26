---
layout: post
title: getBean의 NoSuchBeanDefinitionException 회피하기
description: getBean의 NoSuchBeanDefinitionException 회피하기
date:   2019-09-26 13:00:00 +0900
author: Jeongjin Kim
categories: Spring
tags:	Spring
---

Spring context에서 Bean을 가져오기 위해 `getBean` 메소드를 주로 사용한다.

`getBean` 메소드는 Bean 이름이나 Type 등을 받아서 해당하는 Bean이 있으면 인스턴스를 반환하고 
없으면 `NoSuchBeanDefinitionException` 예외를 발생시킨다.

만약에 A라는 Bean이 있으면 이걸 사용하고, 없으면 B라는 Bean을 사용해야 할 때 어떻게 구현을 해야 할까?
라는 고민에서 이 포스트를 작성하게 되었다.

### 예외는 진짜 예외일 때 사용하라

이펙티브 자바라는 책을 보면 예외는 진짜 예외상황에만 사용하라고 되어있다. 그 이유는 당연하게도 예외는 말 그대로 예외이기 때문이며 예외를 이용하여 제어문을 사용했을 때 몇 배나 느릴 뿐만 아니라 try-catch 구문으로 인해 JVM이 사용할 수 있는 최적화가 제한된다고 한다.

이에 따르면 A라는 Bean이 있으면 사용하고 없으면 B라는 Bean이라는 사용하는 로직을 구현하기 위한 다음과 같은 구현은 지양을 해야 한다.

```java
try {
    bean = applicationContext.getBean("beanA");
} catch (NoSuchBeanDefinitionException e) {
    bean = applicationContext.getBean("beanB");
}
```

그럼 예외를 발생시키지 않는 메소드는 없나 해서 찾아보니 단일 Bean을 반환하는 메소드는 모두 예외를 발생시키고 여러 개 빈을 반환하는 메소드는 예외를 발생하지 않고 빈 컬렉션을 반환한다는 것을 찾았다.

대표적인 메소드가 `getBeansOfType` 이다.
이 메소드는 타입을 파라미터로 받아 Bean 이름을 Key, 객체를 Value로 하는 Map을 반환한다.
반환된 컬렉션 객체의 크기를 조사해서 분기 제어를 하던지, 명시적으로 원하는 Bean 이름으로 가져와서 사용하던지 여러 가지 방법으로 활용할 수 있다.

아래는 Generic 인터페이스를 구현하는 빈들 중에 특정한 타입을 사용하는 Bean을 가져오는 예제이다.

```java
Map<String, AuditConverter> auditConverters = applicationContext.getBeansOfType(AuditConverter.class);

AuditConverter<DbColumnAuditValuePair> auditConverter = null;

for (String key : auditConverters.keySet()) {
    AuditConverter converter = auditConverters.get(key);

    for (Type genericInterface : converter.getClass().getGenericInterfaces()) {
        if (genericInterface instanceof ParameterizedType) {
            Type actualTypeArgument = ((ParameterizedType) genericInterface).getActualTypeArguments()[0];
            if (actualTypeArgument == DbColumnAuditValuePair.class) {
                auditConverter = converter;
            }
        }
    }
}

```