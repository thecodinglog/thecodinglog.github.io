---
layout: post
title: Spring Boot Security  Auto Configuration 끄기
description: Spring Boot Security  Auto Configuration 끄기
date:   2020-01-10 13:00:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags: EnableWebSecurity Security SecurityAutoConfiguration Spring Spring-Boot springboot
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

Spring Boot 웹 애플리케이션에 Security Starter를 추가하면 자동으로 웹 보안이 적용됩니다. 이는 모든 요청에 대한 인증 요청, 기본 사용자 및 패드워드 생성 등이 포함됩니다.

이런 자동설정 역할을 하는 클래스가 _SecurityAutoConfiguration_ 클래스 입니다. 그래서 자동 설정을 끄고자 할 때는 _@SpringBootApplication_ 애노테이션에 _SecurityAutoConfiguration_ 를 읽어 들이지 않도록 설정만 하면 됩니다.

<script src="https://gist.github.com/thecodinglog/2aaadc74e751b941f54d0faac11656dd.js"></script>

또는 properties 파일에 아래 속성을 추가하면 똑같이 동작합니다.

<script src="https://gist.github.com/thecodinglog/1a6ab684a879e4a06a632dff06cac3c1.js"></script>

자동설정을 끄면 커스텀 보안 서비스를 사용해야 한다던가 기존에 있던 시스템과 통합할 때는 유용할 수 있습니다.

그러나 여러가지로 불편한 점이 있는데, 이 상태에서 WebSecurity를 설정하려면 **@EnableWebSecurity** 애노테이션를 필수로 사용해야 하고, Security 설정을 개발자가 직접 모든 것을 다 해야 합니다. 또한 다른 자동 설정 클래스에서 SecurityAutoConfiguration 가 필요하다면 문제가 될 수 있습니다.

그래서 끄는 것 보다는 기본 설정을 무시하고 개발자가 지정한 설정을 우선 사용하도록 하는 것이 대부분의 경우 유용할 것입니다. 스프링 부트는 사용자 정의 클래스를 발견하면 당연하게도 자동 설정보다 우선하여 동작하게 되어있습니다.

아래 샘플 코드를 참조하여 Web Security 기본설정을 커스터마이징 해보세요.

<script src="https://gist.github.com/thecodinglog/3141346d5b7374839d2c582c83a9d28b.js"></script>