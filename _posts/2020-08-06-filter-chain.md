---
layout: post
title: 스프링 세션 동작 원리 - 스프링 세션 어떻게 동작하는 것인가?
description: Spring-Session 동작 원리
date:   2020-08-07 09:00:00 +0900
author: Jeongjin Kim
categories: spring-session
tags:	spring-session spring filter servlet-filter
---

Spring-Session을 사용하면 외부 저장 매체(mysql, Redis 등)를 이용해서 여러 서버의 **Session을 쉽게 동기화** 할 수 있습니다.
실제로 Session 동기화 기능이 있는 컨테이너를 사용하려면 막대한 비용이 들기도 하고 구성하기가 쉽지도 않습니다.
특정 컨테이너 기술에 의존하게 되는 것도 문제입니다. Tomcat 2개로 서비스를 운영하다가 제우스에 서비스를 추가로 올려서 사용할 수 없다는 의미입니다.

Spring-Session의 또 다른 이점은 한 화면에서 **여러 Session 쉽게 구성**할 수 있도록 해줍니다. G-Mail 서비스를 보면 여러 ID를 변경해가면서 사용할 수 있는데 그런 거라고 보시면 됩니다.

이렇게 편리한 기능을 쉽게 사용할 수 있게 해주는 Spring-Session은 어떤 원리로 동작하는지 살펴보겠습니다.

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



# 서블릿 컨테이너에서 필터 동작

Spring-Session은 필터로 동작하기 때문에 서블릿 컨테이너(Tomcat, Wildfly etc.)에서 필터가 어떤 방식으로 동작하는지 먼저 알아야 할 필요가 있습니다.
필터는 어떻게 만들어지고 호출되는지 대표적인 오픈소스 서블릿 컨테이너인 Tomcat의 소스를 살짝 열어서 살펴보았습니다.

서블릿 컨테이너는 지정된 포트(예, 80 포트, 8080 포트)에서 TCP Socket을 리스닝하고 있다가 네트워크로부터 데이터가 들어오면 그때부터 일을 시작합니다.
데이터가 들어온다는 말은 브라우저나 여타 클라이언트에서 HTTP를 이용해서 데이터를 요청했다는 것이겠죠.

소켓으로 들어온 데이터를 Request라는 객체로 잘 감싸서 서블릿을 호출할 때 같이 넘겨줍니다. 이 서블릿을 호출하기 전에 배포기술자, web.xml,에 정의해놓은 필터들을 먼저
실행시킵니다. Tomcat은 **FilterChain 에게 필터들을 순서대로 호출하고 마지막에 서블릿을 호출**하는 것까지 위임합니다.

서블릿 필터는 `doFileter` 메소드를 가진 인터페이스인데 `doFilter` 의 파라미터를 보면 `Request`, `Response`, `FilterChain` 3가지가 있습니다.
즉, Tomcat이 필터 체인을 만들고 필터 실행을 호출할 때 `Request`와 `Response` 객체뿐만 아니라 만들어 놓은 **필터 체인** 까지 같이 넘깁니다.
필터 체인은 스스로가 등록된 필터들을 직접 호출하는 것이 아니라 **등록된 필터가 넘겨받은 필터 체인을 이용해서 그다음 필터를 호출**하게 되어있습니다. 
그래서 필터들의 구현을 보면 항상 필터 호출 코드가 있습니다. **이 코드를 호출하지 않으면 그 뒤에 붙어있는 필터들은 호출되지 않습니다.**

```java
chain.doFilter(request, response);
```

이 호출은 **재귀 호출**처럼 동작하게 되어서 자신 이후에 호출되는 모든 필터 호출이 종료될 때까지 컨텍스트가 남아있게 되고 `doFilter` 호출 이후의 코드는 이어서 실행이 됩니다.
이렇게 하면 필터가 실행되는 스택이 깊어지고 개발자가 필터를 만들 때 여러 가지 고려해야 할 사항들이 많아지지만 모든 필터를 호출하고 난 뒤 뭔가 액션이 필요할 때는 유용합니다.

그림으로 다시 살펴보면 필터 체인이 필터들을 재귀적으로 호출하고 마지막에 서블릿을 실행시키는 형태로 그려볼 수 있겠습니다. Spring 웹을 사용하고 있다면 컨트롤러 역할을 하는 `DispatcherServle#service` 메소드를 호출하겠지요.

<p><img src="/assets/2020-08-06-filter-chain/2020-08-06-filter-chain_125602.png" width='400px'></p>

>Spring부트를 쓰고 있다면 보기는 힘들겠지만 _web.xml_ 에 서블릿 등록한 것을 보면 요청에 매핑된 서블릿이 `DispatcherServlet` 인것을 확인할 수 있습니다.
>```xml
><servlet>
>    <servlet-name>action</servlet-name>
>    <servlet-class>org.springframework.web.servlet.DispatcherServlet
>    </servlet-class>
>    <load-on-startup>1</load-on-startup>
></servlet>
><servlet-mapping>
>    <servlet-name>action</servlet-name>
>    <url-pattern>/</url-pattern>
></servlet-mapping>
>```



# Session

HTTP는 태어났을 때부터 **상태를 유지하지 않는 프로토콜**입니다. 상태를 유지한다는 말은 요청자가 누구인지 식별하고 식별된 요청자와 관련된 정보들을 만들고 지우고 한다는 말입니다.
우리가 주야장천 쓰는 홈페이지는 HTTP로 동작을 하는데 상태관리를 하고 있을까요? 실제 웹 애플리케이션에서는 현재 상태를 유지하지 않고서는 도저히 올바른 서비스를 제공할 수가 없습니다.
현재 요청을 누가 보냈냐에 따라서 어떤 서비스를 제공할지 또는 하지 말지를 정하기도 하고 편의 기능을 제공하기도 합니다. 

그러면 웹 서비스는 상태를 관리하지 않는 HTTP로 동작을 하는데 어떻게 요청자 정보를 관리하고 있을까요? 바로 **Session** 이 그 역할을 하고 있습니다. 
Session 구현을 위해 존재하는 `HttpSession` 인터페이스는 속성을 추가하고 삭제하고 Id를 가져오고 타임아웃 시간을 설정하고 하는 메소드들이 담겨있는 인터페이스입니다.
Tomcat은 클라이언트별로 Session 객체를 만들어서 내부에 보관하고 있다가 다음 요청이 있을 때 참조합니다. 여기서 _Session Tracking_ 이라는 개념이 등장합니다.

# Session Tracking

일반적인 HTTP 요청에는 내가 누구인지에 대한 정보가 없습니다. 받고 싶어 하는 리소스 주소, 파라미터 정도 있을 뿐입니다.
서블릿 컨테이너가 수신한 요청에 누가 보낸 요청인지에 대한 정보, 즉 식별자가 있어야 내부에 저장하고 있는 Session객체를 읽어서 참조할 것입니다.
다시 말하면, 서블릿 컨테이너가 누구인지 **어떻게 식별**하게 하느냐에 대한 이야기가 _Session Tracking Mechanism_ 입니다.

Session을 추적하기 위해서 **Cookie**를 사용하는 게 가장 일반적입니다. 컨테이너에서 요청에 대한 응답을 내릴 때 Session을 만들고 그 SessionID를 Cookie로 내려보내는 것입니다.
클라이언트에 내려간 Cookie는 그 뒤에 이어오는 요청에 덧붙여져서 오기 때문에 컨테이너가 요청을 받았을 때 Cookie에 있는 Session 정보를 이용해서 요청자를 식별할 수 있습니다.
그래서 웹브라우저의 개발자 도구로 현재 페이지의 쿠키목록을 보면 _JSESSIONID_ 가 있는 것입니다. 이 쿠키 키는 Servlet-Spec에 표준 이름으로 정해놓은 것입니다만 Spring-Session도 그렇고 적절히 바꿔서 사용하는 경우가 많습니다. 서버가 Servlet 기술을 사용하고 있다는 것을 숨기려고 하는 목적도 있습니다.
Servlet Container는 web.xml에서 간단하게 변경할 수 있습니다.

```xml
<session-config>
    <cookie-config>
        <name>mySessionId</name>
    </cookie-config>
</session-config>
```

Session 트래킹을 위한 방법은 SSL, URL Rewriting 두 가지가 더 있는데 SSL이야 어차피 요청에 인증정보가 있어야 하므로 식별할 수 있고 URL은 URL에 ID를 직접 넣어주는 방식인데 보안상 매우 취약하므로 특별한 이유가 없으면 사용하면 안 됩니다.
마찬가지로 _session-config_ 요소로 지정할 수 있습니다. 디폴트값은 `COOKIE`입니다.(Java Servlet 3.1 기준)

```xml
<session-config>
    <tracking-mode>
        COOKIE
    </tracking-mode>
</session-config>
```

# 게으른 Session

컨테이너에서 Session 객체를 만들거나 Request에 있는 ID로 Session을 가져오는 비용은 만만하지가 않기 때문에 **요청이 있을 때** 그때 새로 만들거나 찾아옵니다.
요청이 있을 때란 `Filter`, `Servlet` 등에서 `Request` 객체의 `getSession()` 메소드를 **최초로** 호출하는 순간입니다.
Session 트레킹을 위해 COOKIE를 사용한다고 설정을 해놨을 경우, Session을 새로 만들 때 `Response` 객체의 헤더에 "_나 이런 SessionID로 만들어 놨어_" 하고 헤더 하나를 추가합니다. 그 헤더가 `Set-Cookie` 입니다. 이 헤더는 애플리케이션에서 쿠키 생성이 필요할 때 호출하는 `response.addCookie()` 메소드로 추가되는 것입니다. 즉, `addCookie` 메소드는 응답 헤더에 `Set-Cookie` 하나를 추가하는 일을 합니다. 참고로 Tomcat은 별도 설정이 없으면 Session 객체로 `org.apache.catalina.session.StandardSession` 를 사용합니다.


이렇게 Session이 만들어지는 타이밍이 가장 먼저 Session을 요구했을 때이기 때문에 누가 먼저 Session을 점령하느냐가 중요해집니다. 왜 그런지 Spring-Session 필터를 살펴보면서 말씀드리겠습니다.

# Spring-Session 필터

Spring-Session을 설정하려면 필터를 추가해야 합니다. 다시 얘기하지만 Spring-Session은 필터로 동작합니다. 애노테이션을 이용한 설정을 하더라도 자동으로 필터를 추가하는 코드가 들어가 있는 것입니다. 

```xml
<filter>
	<filter-name>springSessionRepositoryFilter</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
	<filter-name>springSessionRepositoryFilter</filter-name>
	<url-pattern>/*</url-pattern>
	<dispatcher>REQUEST</dispatcher>
	<dispatcher>ERROR</dispatcher>
</filter-mapping>
```

그런데 필터 클래스가 `DelegatingFilterProxy` 입니다. 이 Spring 클래스가 _springSessionRepositoryFilter_ 이란 이름을 가진 빈을 찾아서 실행해 줄 건데 그 필터 클래스가 `org.springframework.session.web.http.SessionRepositoryFilter` 입니다. 이 필터는 `Request` 객체로 `getSession()` 메소드를 호출했을 때 _Spring-Session_ 에서 관리하는 위치에서 Session 정보를 가져올 수 있도록 `Request` 객체를 **Warpping** 하고, Session 객체를 `SessionRepository` 를 이용해서 저장하는 일을 합니다. 뭐 크게 하는 일은 없습니다.

`SessionRepository`는 객체로 만들어진 Session을 어딘가에 저장하는 기능을 제공하기 위한 인터페이스인데 이게 어떤 구현체인가에 따라서 JDBC를 이용해서 RDBMS에 Session정보를 저장하느냐, Redis를 이용해서 저장하느냐가 갈립니다. 그래서 Spring-Session을 프로젝트에 반영할 때 _spring-session-jdbc_, _spring-session-redis_ 같은 의존성을 같이 추가하는 것입니다.

이 Request 정보는 필터들을 거치고 Servlet에서 얼마든지 변경이 가능하기 때문에 모든 서비스가 수행되고 난 뒤 최종적으로 그 결과를 저장소에 저장해야합니다. 앞서 필터 체인이 재귀 호출로 동작하고 `doFilter` 이후에 또 다른 코드로 무언가를 할 수 있다고 말씀드렸습니다. 이쯤에서 위에서 그린 그림을 조금 더 자세히 표현을 해보겠습니다.

<p><img src="/assets/2020-08-06-filter-chain/2020-08-06-filter-chain_173534.png" width='400px'></p>

그림에서 보는 것과 같이 필터 체인들이 실행되고 그다음에 서블릿의 `service` 메소드를 호출하는 것이 아닙니다. 재귀 호출처럼 필터가 필터를 호출하고 쭉쭉 내려가다가 더이상 호출할 필터가 없으면 마지막 필터에서 서블릿 서비스를 호출합니다. 서비스가 종료되면 스택을 타고 올라가서 잴 처음에 있었던 Spring-Session 필터의 `doFilter` 메소드가 종료되고 바로 밑에 있는 `commitSession()` 메소드가 수행되면서 Session정보가 최종적으로 저장됩니다. 이때 영속성 저장매체에 저장이 되고 쿠키에도 _SessionID_ 를 내려줍니다. 이때 쓰는 ID가 _SESSIONID_ 입니다. 

만약에 Spring-Session 필터가 제일 먼저 위치하지 않으면 어떻게 될까요? 먼저 수행된 필터가 있다고 가정하고 그 필터에서 Session을 참조하는 코드가 있다고 하면요? 그렇게 되면 Tomcat에 기본 Session 객체인 `StandardSession` 이 만들어지고 이어서 Spring-Session의 Session도 만들어집니다. 어차피 서비스 단에서 사용하는 Session정보는 Spring에서 감싼 정보를 줄 것이기 때문에 크게 상관은 없지만 쓸데없이 그 비싼 Session객체를 두 개나 만드는 꼴이 됩니다. 클라이언트에서 쿠키정보를 확인해보면 JSESSIONID 와 SESSIONID 둘 다 존재하게 됩니다. 그래서 Spring-Session의 필터는 가장 먼저 실행되도록 하는것이 매우 중요합니다.

# 정리하며

이 글을 작성한 이유는 _Spring-Session 이 어떻게 Tomcat이 만들어낸 JSESSIONID 쿠키를 없앨까?_ 라는 질문에서부터 시작했습니다. 그런데 알고 보니까 Spring-Session이 있음으로써 아얘 생성 자체가 안되는 것이었습니다. 만들어진 것을 없애는 것이 아니라 _Session은 게으르다_ 와 _필터와 서비스는 재귀 호출로 실행된다_ 라는 원리를 가지고 위트있게 만들어낸 Spring-Session 프로젝트에 감탄할 수 밖에 없었습니다.


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