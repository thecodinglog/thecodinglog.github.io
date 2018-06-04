---
layout: post
title: Spring MVC 경로 설정
subtitle: Spring MVC 경로설정시 생기는 문제점에 대한 해결
description: Spring MVC 경로설정시 생기는 문제점에 대한 해결
date:   2017-07-31 08:43:59
author: Jeongjin Kim
categories: Spring
tags:	spring mvc
---
### 목표

* 크게 3가지 타입의 요청을 수신해서 처리 결과를 리턴한다.

* UI page

* Ajax 기반 Service

* 전문

* 요청에 Media Type을 포함하여 해당하는 Media type으로 결과를 리턴한다.

### Url

#### UI Page 요청

~/{media type}.page/{page id}

#### Ajax 기반 Service 요청

~/{media type}.service/{service id}

#### 전문 요청

~/{media type}.message/{message id}

### Web Project 구조

* java

  * controllers

* resources

  * contexts : root application contexts for spring
  * mybatis
    * mappers

* webapp

### 설정 파일들

주요 구문 이외는 생략 하였음

#### web.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID" version="2.5">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:contexts/context-*.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/config/springmvc/dispatcher-servlet.xml</param-value>
        </init-param>

        <load-on-startup>1</load-on-startup>
    </servlet>

     <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

#### dispatcher-servlet.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
                http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

    <!-- Enables the Spring MVC @Controller programming model -->
    <mvc:annotation-driven />
    <context:component-scan base-package="controllers" />
    <mvc:default-servlet-handler />

    <bean
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>

</beans>
```

### 콘트롤러들

#### Ajax 기반 Service 요청 콘트롤러

```java
@RestController
public class ServiceWebController {
    @RequestMapping("/{mediaType}.service/{serviceId}")
    public VO serviceRootRequestHandler(@PathVariable("serviceId") String serviceId,
                        @PathVariable("mediaType") String mediaType) {
        ...
        return VO;
    }
}
```

#### UI Page 요청

```java
@Controller
public class PageWebController {
    @RequestMapping("/html.page/{pageId}")
    public String serviceJsonRequestHandler(@PathVariable("pageId") String pageId) {
        return pageId;
    }
}
```

### 이렇게 세팅 했을 때 문제점

WEB-INF 및에 위치하고 있는 View에서 static 자원을 참조하려고 할 때

```
<script src="/js/ui.bootstrap.js"> </script>
```

이렇게 세팅했을 때 절대 위치로 Path가 잡혀서 [http://www.abc.com/js/ui.bootstrap.js에](http://www.abc.com/js/ui.bootstrap.js에) 있는 자원을 가져온다. 만약 Web root가 /newapp/ 이라면 [http://www.abc.com/newapp/js/ui.bootstrap.js](http://www.abc.com/newapp/js/ui.bootstrap.js) 에 있는 자원을 참조하려는 의도였지만 절대 위치로 잡혀있기 때문에 web root와 상관없이 [http://www.abc.com/js/ui.bootstrap.js](http://www.abc.com/js/ui.bootstrap.js) 자원을 참조하려고 하기 때문에 참조 오류가 발생한다.

```
<script src="./js/ui.bootstrap.js"> </script>
```

현재 위치 기준으로 참조하려고 하면 페이지 요청 URL이 [http://www.abc.com/page/pageid](http://www.abc.com/page/pageid) 라면 [http://www.abc.com/page/pageid/js/ui.bootstrap.js](http://www.abc.com/page/pageid/js/ui.bootstrap.js) 가 Static 자원 요청 URL이 되어서 실제 있는 위치와 다르게 된다. 억지로 상대 주소를 맞추고자 하면

```
<script src="../../../js/ui.bootstrap.js"> </script>
```

와 같이 매우 불편하게 위치를 지정할 수 밖에 없다.

### 처리방안

절대 위치로 URL을 지정하되 Context Root를 가져와서 Concat 시키는 방법

#### JSP 예

```java
<script src="<%=request.getContextPath()%>/js/ui.bootstrap.js"> </script>
```

#### Javascript 예

```js
contextRoot = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2));
document.write("<link rel=\"stylesheet\" type=\"text/css\" href=\"" + contextRoot + "/dhtmlx/codebase/fonts/font_roboto/roboto.css\"/>");
```
