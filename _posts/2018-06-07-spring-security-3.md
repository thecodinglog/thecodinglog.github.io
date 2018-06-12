---
layout: post
title: Spring Security Reference 따라하기 3
description: Security Namespace Configuration로 DB 기반 로그인
date:   2018-06-07 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---
##  Security Namespace Configuration 으로 보안 설정하기
이번에는 XML로 보안 설정을 해보자.

gradle을 이용해서 프로젝트를 만들어 보겠다. gradle이 설치되어 있지않다면 [여기](/gradle/2018/05/30/install-gradle-in-windows.html)를 참고하여 설치하자.

### 프로젝트 구조 만들기

```sh
mkdir spring-security-reference2
cd spring-security-reference2

gradle init

mkdir -p src/main/java
mkdir -p src/main/resources
mkdir -p src/test/java
mkdir -p src/test/resources
mkdir -p src/main/webapp/WEB-INF
```
> `gradle init` 에 옵션으로 `--type java-application` 을 넣어주면 _webapp_ 디렉토리를 제외한 나머지는 자동으로 만들어준다.


### 웹 애플리케이션 메타데이타 설정

servlet 3.0 이상이라면 메타데이타를 설정하는 방법은 2가지가 있는데 deployment descriptor인 _web.xml_ 을 _WEB-INF_ 디렉토리에 넣거나,  Annotation 으로 설정 할 수 있다.

우리는 _web.xml_ 을 추가하자.

**src\main\webapp\WEB-INF\web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
		 http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

</web-app>
```

### gradle 설정

build.gradle 파일을 열어서 다음 내용 추가

**build.gradle**

```groovy
plugins {
    id 'java'
    id 'war'
    id "org.akhikhl.gretty" version "2.0.0"
}

gretty{
    servletContainer = 'tomcat8'
    managedClassReload = true
    fastReload = true
    contextPath = '/'
}

repositories {
  jcenter()
}

dependencies {
  providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
  testCompile 'junit:junit:4.12'
}

```

### Sample 페이지 작성

**src\main\webapp\index.html**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    Hello
  </body>
</html>
```

### 실행
```sh
./gradlew appRun
```

http://localhost:8080/ 로 접속하면 Sample page가 잘 열린다.

### 스프링 설정

이제 여기에 Spring 설정과 Security 설정을 추가해보자. 지금부터는 package 도 import 해야하고 의존성도 추가해야하고 이것저것 귀찮은게 많으니 intellij 에서 이 프로젝트를 열자.

#### _web.xml_ 에 스프링 설정 추가

**src\main\webapp\WEB-INF\web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
		 http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:contexts/*.xml</param-value>
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

#### 추가로 필요한 디렉토리는 생성

```
mkdir -p src/main/webapp/WEB-INF/config/springmvc
mkdir -p src/main/resources/contexts
```

#### dispatcher-servlet.xml 만들기

**src/main/webapp/WEB-INF/config/springmvc/dispatcher-servlet.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        ">
    <mvc:annotation-driven/>
    <context:component-scan base-package="cothe.controllers"/>
</beans>
```

#### 컨트롤러 추가

```java
package cothe.controllers;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RestWebController{
    @GetMapping("/status")
    public String status(){
        return "Ok!";
    }
}
```


#### build.gradle 파일에 스프링 의존성 추가

`compile 'org.springframework:spring-webmvc:5.0.5.RELEASE'`


**build.gradle**
```groovy
plugins {
    id 'java'
    id 'war'
    id "org.akhikhl.gretty" version "2.0.0"
}

gretty{
    servletContainer = 'tomcat8'
    managedClassReload = true
    fastReload=true
    contextPath='/'
}

repositories {
  jcenter()
}

dependencies {
  providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
  compile 'org.springframework:spring-webmvc:5.0.5.RELEASE'
  testCompile 'junit:junit:4.12'
}
```

Intellij의 Gradle tool view를 열어서 실행 appRun 실행

![](/assets/spring-security/2018-05-30-spring-security-3-916a65c5.png)

http://localhost:8080/status

![](/assets/spring-security/2018-05-30-spring-security-3-61edd726.png)


### Security 설정

#### Security 설정 파일 추가

**src/main/resources/contexts/security.xml**


```XML
<b:beans xmlns="http://www.springframework.org/schema/security"
         xmlns:b="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">

    <http>
        <intercept-url pattern="/login" access="permitAll()"/>
        <intercept-url pattern="/**" access="hasRole('USER')"/>
        <form-login />
        <logout/>
    </http>

    <authentication-manager>
        <authentication-provider>
            <user-service>
                <user name="jimi" password="password" authorities="ROLE_USER"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>
</b:beans>
```
`<http>`는 `FilterChainProxy`를 만들고 필터 빈을 만든다. `<authentication-provider>`는 `DaoAuthenticationProvider`빈을 만들고 `<user-service>`는 `InMemoryDaoImpl`을 만든다.
`<authentication-provider>`는 인증 관리자가 인증 요청을 처리하기 위해서 사용자 정보를 쓴다는 의미이다. 이걸 여러 인증 소스를 정의하기 여러개를 쓸 수 있다.

설정파일을 추가하고 권한과 사용자 정보를 넣어서 한번 실행해보자.

익숙한 로그인 화면이 뜨는데 로그인 해보면 `PasswordEncoder` 없다고 예외가 발생한다. 인코더를 지정하자.

```xml
<user-service>
    <user name="jimi" password="{noop}password" authorities="ROLE_USER"/>
</user-service>
```
암호에 `NoOpPasswordEncoder`를 사용하겠다는 의미로 접두사 _{noop}_ 를 붙인다. 이전장에서 소개 했듯이 접두사는 어떤 인코더를 사용 할 것인지 `DelegatingPasswordEncoder`에게 알려주는 역할을 한다. 실 프로덕션에서는 절대로 쓰면 안되겠지만 연습할 때 읽기 편하므로 여기서는 이렇게 한다. 실제로 개발할 때는 _**BCrypt**_ 를 사용하자.


### 인증절차 자세히 보기

![](/assets/spring-security/Authentication_uml.jpg)

`AbstractUserDetailsAuthenticationProvider` 의 `authenticate` 메소드에서 `preAuthenticationChecks.check()` 와 `additionalAuthenticationChecks()`, `postAuthenticationChecks.check()` 를 순서대로 호출하면서 인증처리를 한다. 별도로 등록된  `UserDetailsChecker` 빈이 없으면 `DefaultPreAuthenticationChecks` 와 `DefaultPostAuthenticationChecks`를 기본 사용한다. pre, post 인증 체커는 `UserDetails` 가 Lock인지, Enabled인지 Expired 되지 않았는지 체크하고, Credential이 Expired 되지 않았는지 각각 체크한다. 여기서 확장포인트는 **`additionalAuthenticationChecks()`,  `retrieveUser()`** 이다.

`DaoAuthenticationProvider` 는 이 두 메소드를 구현했는데 주입받은 `UserDetailsService`를 이용하여 `UserDetails`를 가져오고, 입력받은 Password와 `UserDetails`의 Password를 비교하여 인증한다.

**따라서 사용자 정보를 가져오는 `UserDetailsService` 를 구현하여 빈으로 등록하면 현지 시스템과 연동된 인증을 할 수 있다.**

### 화면별 인증

특정 페이지는 시큐리티 필터를 생략하고 다른 요청에 대해서는 시큐리티를 탈수 있도록 변경해보자.

#### Security Filter chain 추가
**src/main/resources/contexts/security.xml**

```xml
<http pattern="/index.html*" security="none"/>
```
시큐리티 설정파일에 index.html* pattern 으로 온 요청은 무시하도록 한 출을 추가한다.

#### Default Servlet Handler 추가
**src/main/webapp/WEB-INF/config/springmvc/dispatcher-servlet.xml**
```xml
<mvc:default-servlet-handler />
```
스프링의 기본 핸들러 매핑전략으로 매칭되는 컨트롤러를 찾고 없으면 디폴트 서블릿으로 요청을 넘기도록 한다.

#### Sample page 추가

**src/main/webapp/sample.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Sample page
</body>
</html>
```

_index.html_ 을 요청하면 시큐리티 필터를 스킵하고 요청한 페이지가 열린다.

반면에 _sample.html_ 을 요청하면 시큐리티 필터에 의해서 걸러지고 로그인 페이지로 넘어가게 된다. 로그인을 하면 최초 요청했던 _sample.html_ 이 열린다.


### Custom Login Page 로 인증하기

#### security 설정
**src/main/resources/contexts/security.xml**

```xml
<http>
    <intercept-url pattern="/loginPage" access="permitAll()"/>
    <intercept-url pattern="/**" access="hasRole('USER')"/>
    <form-login login-page="/loginPage"/>
    <logout/>
</http>
```

`<form-login>` 에 _login-page_ 속성으로 로그인 페이지를 지정할 수 있다. 최상단에 로그인 페이지를 모두 허용으로 바꾸지 않으면 로그인 무한 반복으로 페이지 요청이 생기므로 꼭 설정해야 한다.

#### Page Controller 추가

```java
package cothe.controllers;

@Controller
public class WebController {
    @GetMapping("/loginPage")
    public String login() {
        return "login";
    }

    @GetMapping("/home")
    public String home() {
        return "home";
    }
}

```
_/loginPage_ 로 요청을 받으면 _login.jsp_ 로 넘기도록 넣고, 테스트 할 페이지로 _home.jsp_ 를 추가했다.

#### Custom login page 와 home 작성

**src/main/webapp/WEB-INF/view/home.jsp**

```html
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>home</h1>
<c:url value="/logout" var="logoutUrl"/>
<form action="${logoutUrl}" method="post">
    <input type="hidden"
           name="${_csrf.parameterName}"
           value="${_csrf.token}"/>
    <button type="submit" class="btn">Log out</button>
</form>
</body>
</html>
```

**src/main/webapp/WEB-INF/view/login.jsp**

```html
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>Custom login page</h1>
<c:url value="/login" var="loginUrl"/>
<form action="${loginUrl}" method="post">
    <c:if test="${param.error != null}">
        <p>
            Invalid username and password.
        </p>
    </c:if>
    <c:if test="${param.logout != null}">
        <p>
            You have been logged out.
        </p>
    </c:if>
    <p>
        <label for="username">Username</label>
        <input type="text" id="username" name="username"/>
    </p>
    <p>
        <label for="password">Password</label>
        <input type="password" id="password" name="password"/>
    </p>
    <input type="hidden"
           name="${_csrf.parameterName}"
           value="${_csrf.token}"/>
    <button type="submit" class="btn">Log in</button>
</form>
</body>
</html>
```
#### 로그인 해보기
![](/assets/spring-security/2018-05-30-spring-security-3-a668714d.png)

![](/assets/spring-security/2018-05-30-spring-security-3-ec34bbe5.png)

홈 화면도 잘 뜨고 로그아웃도 잘 된다.

![](/assets/spring-security/2018-05-30-spring-security-3-e9f99358.png)

패스워드를 틀리면 틀렸다고 짹짹
![](/assets/spring-security/2018-05-30-spring-security-3-81506611.png)


### JDBC 사용해서 로그인하기

**src/main/java/cothe/controllers/WebController.java**

```java
@Controller
public class WebController {
    @GetMapping("/loginPage")
    public String login() {
        return "login";
    }

    @GetMapping("/home")
    public String home() {
        return "home";
    }
}
```

**src/main/resources/contexts/context-datasource.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/security_db"/>
        <property name="username" value="secuser"/>
        <property name="password" value="sec1234"/>
    </bean>
</beans>
```

**src/main/resources/contexts/security.xml**

```xml
<b:beans xmlns="http://www.springframework.org/schema/security"
         xmlns:b="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">


    <http pattern="/index.html*" security="none"/>

    <http>
        <intercept-url pattern="/loginPage" access="permitAll()"/>
        <intercept-url pattern="/**" access="hasRole('USER')"/>
        <form-login login-page="/loginPage"/>
        <logout/>
    </http>

    <authentication-manager>
        <authentication-provider user-service-ref='myUserDetailsService'/>
    </authentication-manager>

    <b:bean id="myUserDetailsService"
            class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
        <b:property name="dataSource" ref="dataSource"/>
    </b:bean>
</b:beans>
```

**src/main/webapp/WEB-INF/config/springmvc/dispatcher-servlet.xml**

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/view/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

**build.gradle**

```groovy
compile group: 'mysql', name: 'mysql-connector-java', version: '5.1.46'
compile group: 'org.apache.commons', name: 'commons-dbcp2', version: '2.3.0'
```
>mysql-connector-java가 6 이상 버전에서는 타임존이 안맞다고 오류가 발생한다. database 의 타임존을 맞춰주던지 그 이하 버전을 사용한다.



**users** table
![](/assets/spring-security/2018-05-30-spring-security-3-2775061a.png)
inMemory 를 활용했을 때와 마찬가지로 인코딩을 사용하지 않도록 했다.

**authorities** table
![](/assets/spring-security/2018-05-30-spring-security-3-8c58b9fe.png)

이렇게 spring-boot 와 Namespace Configuration를 쓰는 방법으로 로그인 로그아웃 기능을 구현하였다.

### Default AccessDecisionManager

_namespace configuration_ 을 사용하면 `AccessDecisionManager` 의 디폴트 인스턴스가 자동으로 등록된다. 이게 메소드 호출이나 intercept-url에 기반한 웹 URL 액세스, protect-pointcut 등에 대하여 접근 제어를 한다.

기본 전략은 `RoleVoter` 및 `AuthenticatedVoter`와 함께 `AffirmativeBased` `AccessDecisionManager`를 사용하는 것이다.
