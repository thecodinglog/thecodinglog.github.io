---
layout: post
title: Spring-boot가 아닌 프로젝트를 Spring-boot-test로 테스트하기
description: Spring-boot가 아닌 프로젝트를 Spring-boot-test로 테스트하기 @SpringBootTest for a non-spring-boot application
date:   2018-09-27 16:30:00 +0900
author: Jeongjin Kim
categories: Spring-boot
tags:	spring spring-boot 스프링부트 스프링부트테스트 spring-boot-test
---
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>


Spring을 사용하는 라이브러리를 구현하고 **_spring-boot-test_** 로 테스트하고자 한다.

문제는 스프링부트 테스트를 만들어도 정상적으로 DI를 받을 수가 없다. 아래와 같이 컴포넌트를 만들고 테스트를 한번 만들어보자.

```java
package cothe.springboottestfornormalproject.sample;

import org.springframework.stereotype.Component;

@Component
public class User {
    private String name;
    public User() {
        this.name = "default";
    }

    public String getName() {
        return name;
    }
}
```

```java
package cothe.springboottestfornormalproject.sample;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserTest {
    @Autowired
    User user;

    @Test
    public void defaultName(){
        assertEquals(user.getName(), "default");
    }
}
```

간단하게 컴포넌트를 테스트하는 코드를 만들면 `User` 를 자동연결 할 수 없다고 IDE에서 미리 에러를 내밷는다.

![](/assets/2018-09-27-spring-boot-test/2018-09-27-spring-boot-test_165716.png)

테스트하고자하는 애플리케이션이 스프링부트 애플리케이션이면 main 메서드가 있는 클래스를 `@SpringBootApplication` 이라 선언해서 쉽게 테스트를 만들텐데 라이브러리 모듈은 main 메소드가 **없을 수** 있다. `@SpringBootApplication` 애노테이션은 별도 설정이 없으면 클래스패스 하위에 있는 클래스들을 모두 탐색해서 스프링 컴포넌트들을 찾는 등 많은 역할을 한다. 다시 말해서 스프링부트가 어디서부터 봐야하는지 설정하지 않아 에러가 발생하는 것이다.

이 역할을 하는 클래스를 **테스트 모듈**에 만들고 **컴포넌트를 스캔하는 Base를 지정**해서 이와 같은 문제를 해결 할 수 있다.

```java
package cothe.springboottestfornormalproject;

import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication(scanBasePackages = "cothe.springboottestfornormalproject.*")
public class SpringBootTestForNormalProjectApplicationTests {
}
```

