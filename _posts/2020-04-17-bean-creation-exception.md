---
layout: post
title: Error creating bean with name 'dataSource'
description: 
date:   2020-04-17 08:00:00 +0900
author: Jeongjin Kim
categories: spring springboot
tags:	spring boot exception
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


스프링부트 프로젝트를 만들고 springboot-starter-jpa 를 추가하고 바로 실행하면 바로 예외가 발생합니다.

```plain
cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dataSource' defined in class path resource
```

`datasouce` bean 정의를 찾을 수 없다는 예외인데 이유는 당연하게도 jpa를 사용하기 위한 datasource 정보가 없어서 이겠지요.

속성파일 `application.properties` 에 아래 속성을 정의하시면 됩니다.

```plain
spring.datasource.url
spring.datasource.username
spring.datasource.password
spring.datasource.driver-class-name
```

