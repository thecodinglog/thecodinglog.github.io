---
layout: post
title: SpringSecurity FilterChain이 만들어지는 과정 살펴보기
description: SpringSecurity FilterChain이 만들어지는 과정 살펴보기
date:   2020-04-28 09:00:00 +0900
author: Jeongjin Kim
categories: spring
tags:	SpringSecurity Spring Web SpringWeb
---

Spring Security는 **표준 서블릿 필터**를 기반으로합니다. 내부적으로 서블릿 또는 다른 서블릿 기반 프레임 워크 (예 : Spring MVC)를 사용하지 않으므로 특정 웹 기술과은 의존성은 없습니다. `HttpServletRequest`와 `HttpServletResponse`를 사용하고, 브라우저, 웹 서비스 클라이언트, HttpInvoker 또는 AJAX 애플리케이션에서 등 어떤 종류의 클라이언트던지 상관없이 동작합니다.

스프링 시큐리티는 서비스 설정(Configuration)에 따라 필터를 내부적으로 구성합니다. 각 필터는 각자 역할이 있고 필터 사이의 종속성이 있으므로 **순서**가 중요합니다. XML Tag를 이용한 네임스페이스 구성을 사용하는 경우 **필터가 자동으로 구성**됩니다. 네임스페이스 구성이 지원하지 않는 기능을 써야하거나 커스터마이징된 필터를 사용하야 할 경우 명시적으로 빈을 등록 할 수는 있습니다.

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


## DelegatingFilterProxy

스프링 시큐리티에서 필터 클래스는 애플리케이션 컨텍스트에서 정의된 스프링 빈이므로 스프링의 풍부한 의존성 주입 기능과 라이프 사이클 인터페이스를 활용할 수 있습니다. Spring의 `DelegatingFilterProxy`는 `web.xml`과 애플리케이션 컨텍스트 사이의 링크를 제공합니다. `DelegatingFilterProxy` 자체는 스프링 시큐티리가 가지고 있는 필터는 아닙니다.

```xml
<filter>
  <filter-name>springSecurityFilterChain</filter-name>
  <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
  <filter-name>springSecurityFilterChain</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

실질적으로 서블릿 필터는 `DelegatingFilterProxy` 하나이며 이 클래스는 스프링 시큐리티의 실제적인 구현을 가지고 있지는 않습니다. `DelegatingFilterProxy`가 하는 일은 **Spring의 애플리케이션 컨텍스트에서 얻은 Filter Bean를 대신 실행하는 것**입니다. 이를 통해 Bean은 Spring 웹 애플리케이션 컨텍스트 라이프 사이클 지원 및 구성 유연성을 활용할 수 있습니다. **Bean은 javax.servlet.Filter를 구현해야하며 filter-name 요소에 정의된 이름과 동일한 이름의 id을 가져야합니다**. 예를 들면 아래와 같이 정의되어 있어야 합니다.

```
<bean id="myFilter" class="org.springframework.security.web.FilterChainProxy"/>
```

왜냐하면 `DelegatingFilterProxy`가 초기화 될때 자신의 이름과 같은 id를 가진 Bean을 찾아서 대리자로 등록하기 때문입니다.

전체적인 `DelegatingFilterProxy` 구성을 json 형태로 정리해보면 아래와 유사할 것입니다.

```json
{
  "ServletFilterChain": {
    "servletFilters": [
      {
        "DelegatingFilterProxy": {
          "FilterChainProxy": {
            "securityFilterChainList": [
              {
                "SecurityFilterChain": [
                  {
                    "SpringSecurityFilter": "SecurityContextPersistenceFilter"
                  },
                  {
                    "SpringSecurityFilter": "ConcurrentSessionFilter"
                  },
                  {
                    "SpringSecurityFilter": "UsernamePasswordAuthenticationFilter"
                  }
                ]
              }
            ]
          }
        }
      }
    ]
  }
}
```

`ServletFilterChain`은 `web.xml` 에 정의된 filter 갯수대로 _servletFilters_ 를 생성할 것입니다. 앞선 필터 선언에 의하면 `DelegatingFilterProxy` 필터 하나가 추가될 것입니다. `DelegatingFilterProxy`는 자신의 이름과 동일한 id를 가진 Bean을 Application Context에서 가져와 대리자로 등록합니다. 위 경우에는 `FilterChainProxy` 가 대리자로 등록되었습니다. `FilterChainProxy` 는 `SecuriyFilterChain` 리스트를 가지고 있습니다. `SecurityFilterChain` 은 네임스페이스 설정에서 `<http>` 요소당 하나가 만들어진다고 보면됩니다. `SecurityFilterChain` 은 실제 일을 하는 스프링시큐리티 리스트를 가지고 있습니다. 이 목록 등록된 필터가 순서대로 호출되면서 스프링시큐리티 기능이 동작을 합니다.

## FilterChainProxy

Spring Security의 웹 인프라는 `FilterChainProxy` 인스턴스에 위임하여 사용해야합니다. 보안 필터는 단독으로 사용해서는 안됩니다. 이론적으로 application context 파일에 필요한 각 Spring Security 필터 Bean을 선언하고 각 필터에 대해 해당 `DelegatingFilterProxy` 항목을 `web.xml`에 순서대로 추가하여 사용할 수 있지만 번거롭고 복잡합니다.

`FilterChainProxy`를 사용하면 web.xml 에는 단일 항목만 추가하면서 웹 보안 Bean 관리를 위한 애플리케이션 컨텍스트 파일을 사용할 수 있습니다. `FilterChainProxy`는 `DelegatingFilterProxy`을 사용하여 연결합니다.

```xml
<bean id="springSecurityFilterChain" class="org.springframework.security.web.FilterChainProxy">
<constructor-arg>
    <list>
    <sec:filter-chain pattern="/restful/**" filters="
        securityContextPersistenceFilterWithASCFalse,
        basicAuthenticationFilter,
        exceptionTranslationFilter,
        filterSecurityInterceptor" />
    <sec:filter-chain pattern="/**" filters="
        securityContextPersistenceFilterWithASCTrue,
        formLoginFilter,
        exceptionTranslationFilter,
        filterSecurityInterceptor" />
    </list>
</constructor-arg>
</bean>
```
네임 스페이스 요소 `filter-chain` 은 응용 프로그램에 필요한 보안 필터 체인을 편리하게 설정하는데 사용합니다. 필터 요소에 지정된 Bean 이름으로 작성된 필터 목록에 특정 URL 패턴을 맵핑하고 `SecurityFilterChain` 유형의 Bean에서 이들을 결합합니다. pattern 속성은 Ant 경로를 사용하며 가장 구체적인 URI를 먼저 정의해야합니다. 런타임시 `springSecurityFilterChain`는 현재 웹 요청과 일치하는 첫 번째 URI 패턴을 찾고 filters 속성으로 지정된 필터 Bean 목록이 해당 요청에 적용됩니다. 필터는 정의된 순서대로 호출되므로 특정 URL에 적용되는 필터 체인을 완전히 제어 할 수 있습니다.


필터 체인에 두 개의 `SecurityContextPersistenceFilter`가 선언되었음을 알 수 있습니다. (ASC는 `SecurityContextPersistenceFilter`의 속성 인 `allowSessionCreation`의 약자입니다). RESTful 서비스는 요청에 jsessionid를 제공하지 않으므로 이러한 사용자 에이전트에 대해 HttpSession을 생성할 필요가 없습니다. 확장성이 필요한 대용량 응용 프로그램이있는 경우 위에 표시된 방법을 사용하는 것이 좋습니다. 더 작은 응용 프로그램의 경우 단일 `securityContextPersistenceFilter` (기본 `allowSessionCreation`을 `true`로 사용)를 사용하면 충분합니다.

`FilterChainProxy`는 구성된 필터에서 표준 필터 라이프 사이클 메소드를 호출하지 않습니다. 다른 Spring Bean과 마찬가지로 Spring의 애플리케이션 컨텍스트 라이프 사이클 인터페이스를 대안으로 사용하는 것이 좋습니다.

`springSecurityFilterChain` 네임스페이스 구성을 사용하여 웹 보안을 설정하는 방법을 살펴보면 이름이 "**springSecurityFilterChain**"인 `DelegatingFilterProxy`를 사용했습니다. 이제 이것이 네임스페이스에 의해 생성 된 `FilterChainProxy`의 이름임을 알 수 있습니다. 따라서 **web.xml에** **DelegatingFilterProxy 를 등록할 때 이름을** **springSecurityFilterChain 으로 지정해야** **DelegatingFilterProxy와 FilterChainProxy를 연결할 수 있습니다.**

#### 필터 체인 무시하기

필터 Bean 목록을 제공하는 대신 filters = "none" 속성을 사용할 수 있습니다. 이것은 보안 필터 체인에서 요청 패턴을 완전히 생략합니다. 이 경로와 일치하는 항목에는 인증 또는 권한 부여 서비스가 적용되지 않으며 자유롭게 액세스 할 수 있습니다. 요청 중에 SecurityContext 컨텐츠의 컨텐츠를 사용하려면 보안 필터 체인을 통과해야합니다. 그렇지 않으면 SecurityContextHolder가 채워지지 않고 내용이 널이됩니다