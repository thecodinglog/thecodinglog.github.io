---
layout: post
title: Spring Security Reference 따라하기 2
subtitle: Spring-boot와 Spring-security로 DB 기반 로그인
description: Spring-boot와 Spring-security로 DB 기반 로그인
date:   2018-05-25 17:30:00 +0900
author: Jeongjin Kim
categories: Spring
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---

계속해서 레퍼런스를 따라해보자. OAuth 는 일단 넘어가고 [5.8 Authentication](https://docs.spring.io/spring-security/site/docs/5.0.5.RELEASE/reference/htmlsingle/#jc-authentication) 부터 다시 보자.

### Spring-security Sample 실행 해 보기
앞선 예제에서 인메모리 방식으로 사용자를 등록했다. 이걸 JDBC 로 사용자를 가져오는 방식으로 바꿔보자.
레퍼런스에서는
[여기](https://github.com/spring-projects/spring-security/tree/master/samples/javaconfig/jdbc)에 샘플을 제공한다고 되어 있어 있는데 실행해보려면 어차피 전체 프로젝트가 필요하니까 스프링시큐리티 프로젝트를 통째로 clone받자.

git bash에서 다운받기 원하는 디렉토리로 가서 Clone 받고

```sh
git clone git@github.com:spring-projects/spring-security.git
```

다 다운받으면 디렉토리로 가서
```sh
./gradlew install
```
해서 필요한 jar를 local maven cache에 담도록하고

```sh
./gradlew build
```
빌드를 하고 프로젝트를 오픈을 하면 한참 또 인텔리제이가 빌드를 한다.

한참 기다리면

![](/assets/spring-security/2018-05-25-spring-security-2-81176c35.png)

모듈들이 이쁘게 세팅된다.

이제 레퍼런스에서 알려준 모듈을 실행하기 위해 실행 설정을 해야한다.
Run->Edit Configurations 에서 실행하거나 오른쪽 위에 드롭다운 박스에서 실행

![](/assets/spring-security/2018-05-25-spring-security-2-52bc14bc.png)

![](/assets/spring-security/2018-05-25-spring-security-2-070af404.png)

Tomcat Server에 Local 선택

![](/assets/spring-security/2018-05-25-spring-security-2-f53a7c05.png)

적절한 이름을 바꾸고 Deployment 탭에서 Arifact 누르면 가용 Arifact들이 쭉 나오는데 우리가 실행할 것을 선택한다.

![](/assets/spring-security/2018-05-25-spring-security-2-6deff2d9.png)

실행을 하면 또 한참 빌드…

빌드가 다 끝나면

![](/assets/spring-security/2018-05-25-spring-security-2-42f1e921.png)

많이 봤던 화면이 뜨고
소스코드에 보면 사용자를 user / password 로 등록하는 부분이 있는데 그 계정으로 로그인

![](/assets/spring-security/2018-05-25-spring-security-2-3363db08.png)

음… 소스를 보면 그 모듈에는 별다른 view도 없고 여튼 뭐 아무것도 없는데 이게 어디서 세팅되서 처리되는 것인지 궁금하지만 나중에 보기로 하고 jdbc 설정부분만 보면

```java
@Autowired
private DataSource dataSource;

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
  // ensure the passwords are encoded properly
  UserBuilder users = User.withDefaultPasswordEncoder();
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .withDefaultSchema()
      .withUser(users.username("user").password("password").roles("USER"))
      .withUser(users.username("admin").password("password").roles("USER","ADMIN"));
}
```

### JDBC 인증방법으로 변경하기
앞선 코드에서 jdbc 설정부분을
`cothe.springsecurityreference.config` 패키지의
`WebSecurityConfig` 클래스에 넣으면 dataSource를 Autowired 할 수 없다고 뜬다.

왜 그렇지 또 찾아 봐야 겠다.

현재 까지 `WebSecurityConfig` Class 소스 코드

```java
package cothe.springsecurityreference.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import javax.sql.DataSource;

@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter implements WebMvcConfigurer {
    @Autowired
    private DataSource dataSource;

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        User.UserBuilder users = User.withDefaultPasswordEncoder();
        auth
                .jdbcAuthentication()
                .dataSource(dataSource)
                .withDefaultSchema()
                .withUser(users.username("user").password("password").roles("USER"))
                .withUser(users.username("admin").password("password").roles("USER", "ADMIN"));
    }

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withDefaultPasswordEncoder().username("user").password("password").roles("USER").build());
        return manager;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login")
                .permitAll();
    }
}
```

먼저 _application.properties_ 파일을 yml로 변경하고 기존에 등록했던 jsp의 prefix, suffix도 yml 문법에 맞게 고친다.
datasource와 jpa관련 옵션들도 넣어준다. db는 mysql을 설치하여 사용할 것이기 때문에 url도 그에 맞춘다.

**application.yml**

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create
  datasource:
    url: jdbc:mysql://localhost:3306/security_db
    username: secuser
    password: sec1234
```

_build.gradle_ 에 jpa와 database 관련 의존성도 넣어준다.

**build.gradle**
```groovy
compile('org.springframework.boot:spring-boot-starter-security')
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.springframework.boot:spring-boot-starter-data-jpa')
compile('mysql:mysql-connector-java')

compile('javax.servlet:jstl')
compile('org.apache.tomcat.embed:tomcat-embed-jasper')

testCompile('org.springframework.boot:spring-boot-starter-test')
testCompile('org.springframework.security:spring-security-test')
```

엇. spring-boot-starter-data-jpa 의존성을 추가하니 Autowired 오류는 없어졌다.

이렇게 해서 실행해보면 아래와 같이 문법오류가 났다고 뜬다.

```
Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'varchar_ignorecase(50) not null primary key,password varchar_ignorecase(500) not' at line 1
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:1.8.0_144]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62) ~[na:1.8.0_144]
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[na:1.8.0_144]
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423) ~[na:1.8.0_144]
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:425) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.Util.getInstance(Util.java:408) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:944) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3976) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3912) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2530) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2683) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2482) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2440) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.StatementImpl.executeInternal(StatementImpl.java:845) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.mysql.jdbc.StatementImpl.execute(StatementImpl.java:745) ~[mysql-connector-java-5.1.46.jar:5.1.46]
	at com.zaxxer.hikari.pool.ProxyStatement.execute(ProxyStatement.java:95) ~[HikariCP-2.7.9.jar:na]
	at com.zaxxer.hikari.pool.HikariProxyStatement.execute(HikariProxyStatement.java) ~[HikariCP-2.7.9.jar:na]
	at org.springframework.jdbc.datasource.init.ScriptUtils.executeSqlScript(ScriptUtils.java:473) ~[spring-jdbc-5.0.6.RELEASE.jar:5.0.6.RELEASE]
	... 51 common frames omitted


Process finished with exit code 1

```

내가 알기로는 스프링부트에서 자동으로 어떤 DB인지 판단해서 그 DB에 맞는 sql을 실행시키는 것으로 알고 있는데 실행된 쿼리는 mysql 쿼리가 아니라서 오류가 났다.
스프링부트가 설치된 dbms가 mysql이라는 것을 알아채지 못해서 그런것일까?
명시적으로 application.yml에 database를 지정해서해도 마찬가지였다.

조금더 자세한 정보를 얻기 위해서 logback.xml 설정파일을 먼저 추가했다.

![](/assets/spring-security/2018-05-25-spring-security-2-bd74be07.png)

**logback.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
  <encoder>
    <pattern>[%d{yyyy-MM-dd HH:mm:ss}][%-5level][%logger{36}] - %msg%n</pattern>
  </encoder>
  </appender>

  <root level="info">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

새롭게 추가한 인증부분에 브레이크 포인트를 걸고 디버깅을 해봤다.
![](/assets/spring-security/2018-05-25-spring-security-2-20fc2b69.png)
일단 dataSource는 이상없이 가져온 것 같고
![](/assets/spring-security/2018-05-25-spring-security-2-4b8e3410.png)

![](/assets/spring-security/2018-05-25-spring-security-2-108f20d3.png)
dataSource도 잘 세팅되고

![](/assets/spring-security/2018-05-25-spring-security-2-8109c927.png)
응? withDefaultSchema 메소드에서 하드코딩된 스크립트 리소스를 가져오네. 저 스크립트가 어떤 내용인지 보면

![](/assets/spring-security/2018-05-25-spring-security-2-887faef0.png)

아까 로그에서 못 그 쿼리문이 여기에 저장되어 있었네...

아.. 저 `withDefaultSchema` 펑선이 설치된 db가 뭔지 보고 인증관리에 필요한 ddl 문법을 자동으로 선택해서 실행하는 것인 줄 알았는데, 이렇게 까보니 그냥 기본으로 지원하는 dbms(H2 인가?) 의 ddl 만 실행되게 되어 있었군!

그러고 나서 withUser 메소드로 사용자를 넣어주고 하는 그런 의미였네...

ok 그럼

```java
withDefaultSchema()
.withUser(users.username("user").password("password").roles("USER"))
.withUser(users.username("admin").password("password").roles("USER", "ADMIN"));
```

요 3줄을 지우면 잘 실행될 것 같은 느낌이 드므로 지우고 실행.

ㅇㅋ 일단 오류는 나지 않고 실행은 된다. 하지만 분명 설정에 create 옵션을 줬는데 자동으로 테이블이 만들어진 것 같지는 않다.
![](/assets/spring-security/2018-05-25-spring-security-2-c15dd114.png)

필요한 테이블을 생성하고 데이터를 넣은 다음 다시 시도하자

```sql
create table users(
	username varchar(50) not null primary key,
	password varchar(50) not null,
	enabled boolean not null
);
create table authorities (
	username varchar(50) not null,
	authority varchar(50) not null,
	constraint fk_authorities_users foreign key(username) references users(username)
);
create unique index ix_auth_username on authorities (username,authority);

insert into users(username, password, enabled) values('user','password',1);
```
다시 로그인 시도를 해보면
![](/assets/spring-security/2018-05-25-spring-security-2-45c5e80f.png)

로그인이 안되고 로그에

```
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
	at org.springframework.security.crypto.password.DelegatingPasswordEncoder$UnmappedIdPasswordEncoder.matches(DelegatingPasswordEncoder.java:238)
	at org.springframework.security.crypto.password.DelegatingPasswordEncoder.matches(DelegatingPasswordEncoder.java:198)
	at org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration$LazyPasswordEncoder.matches(AuthenticationConfiguration.java:289)
	at org.springframework.security.authentication.dao.DaoAuthenticationProvider.additionalAuthenticationChecks(DaoAuthenticationProvider.java:86)
	at org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider.authenticate(AbstractUserDetailsAuthenticationProvider.java:166)
	at org.springframework.security.authentication.ProviderManager.authenticate(ProviderManager.java:174)
	at org.springframework.security.authentication.ProviderManager.authenticate(ProviderManager.java:199)
	at org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter.attemptAuthentication(UsernamePasswordAuthenticationFilter.java:94)


  생략

```

`PasswordEncoder` 패스워드 인코더가 없다고 뜬다. 해결 방법을 알아보기 전에 전반적인 인증 구조를 먼저 살펴보자.

### 스프링 시큐리티 아키텍처

웹 요청이 오면 필터에서 `AuthenticationManager` 구현체를 가져와서 `authenticate()` 메소드를 호출을 하는것으로 인증을 시도한다.

예제에서는 `UsernamePasswordAuthenticationFilter`를 사용하는데 이건 http 설정에서
 `formLogin()`을 설정하면 `org.springframework.security.config.annotation.web.configurers.FormLoginConfigurer` 에서 `UsernamePasswordAuthenticationFilter` 를 사용하여 초기화 한다.

`AuthenticationManager`는 `AuthenticationManagerBuilder`의 `jdbcAuthentication()`을 호출하면서 설정된다. 메소드에서 `JdbcUserDetailsManager` 를 생성하여 설정하는데 이 Manager에는 database를 조회화고 업데이트하는 쿼리가 저장되어 있다.

`AuthenticationManager`는 스프링시큐리티의 인증을 위한 핵심 인터페이스인데 `authenticate()` 메소드 하나를 가지고 있다. 이것의 구현체 중 `ProviderManager`가 대표적인데 이 클래스는 내부적으로 `AuthenticationProvider` Interface를 리스트로 유지하고 있고 이 Interface의 대표적인 구현체가 `DaoAuthenticationProvider`이다. `AuthenticationProvider` Interface는 `authenticate()` 와 `supports()` 메소드를 가지고 있다.

`DaoAuthenticationProvider`는 `JdbcUserDetailsManager`를 통해 DB의 사용자 정보를 가지고 와서 인증처리를 한다.

`JdbcUserDetailsManager`는 `UserDetailsManager`의 구현체이다. 이 인터페이스는 새 user를 만들고 기존 user를 업데이트 할 수 있는 메소드를 `UserDetailsService` 인터페이스에서 확장한 것이다. `UserDetailsService는` 사용자 정보를 가져오는 핵심 Interface이다.


Configure에 별도로 Bean을 등록하지 않으면 `DelegatingPasswordEncoder`가 기본 인코더로 사용된다. 이 인코더는 내부에 다른 PasswordEncoder를 가지고 있을 수 있는 필드가 있고, `DaoAuthenticationProvider`가 생성되면서 `DelegatingPasswordEncoder`에 적용 가능한 `Encoder`들을 등록한다. 어떤 Encoder를 쓸지는 database에 저장된 password의 prefix {Encoder명} 를 보고 결정한다. 기본값은 _bcrypt_ 이다.

### 패스워드 인코더 설정

DB에 인코딩된 값을 넣기가 힘드니까 인코딩 없이 인증을 하도록 `NoOpPasswordEncoder`를 등록하자.

```java
cothe.springsecurityreference.config.WebSecurityConfig

    @Bean
    public PasswordEncoder noOpPasswordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }
```

실행해보면
로그인 화면이 뜨고
![](/assets/spring-security/2018-05-25-spring-security-2-071ab544.png)

로그인 하면 정상적으로 처리 된다.
![](/assets/spring-security/2018-05-25-spring-security-2-2da8253b.png)
