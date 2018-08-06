---
layout: post
title: Spring Security로 Security 서비스 구축하기 1
description: Spring Security로 인증
date:   2018-07-31 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---

업무 시스템든 웹 서비스든 접근권한 관리 모듈은 필수 구성요소이다. 스프링 시큐리티는 인증과 접근권한 체크를 할 수 있는 클래스가 개발되어 있다. 그러나 실 도메인에서 사용하는 인증 방법과 접근권한체크 방법은 다양하기 때문에 가장 일반적인 방법으로 처리 할 수 있도록 구현되어있다.

여기서는 스프링시큐리티에 개발된 클래스들을 기반으로 사용자관리, 접근권한 관리를 할 수 있는 서비스를 만든다.

# JPA와 JDBC로 데이터 가져오는 실습
프로젝트 만들기
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134112.png)
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134205.png)


Core에 Security와 Lombok을 선택한다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134231.png)

SQL에 JPA, MySql, JDBC를 선택한다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134307.png)

그 뒤는 기본값으로 해서 프로젝트를 생성한다.

프로젝트가 생성됐으니 데이터베이스와 상호작용하는 코드를 간단하게 만들어보자. 데이터베이스는 MySql 8.0 을 설치했다.

전체 프로젝트 구조
```
.
├── main/
│   ├── java/
│   │   └── cothe/
│   │       └── security/
│   │           ├── SecurityApplication.java
│   │           └── core/
│   │               ├── domain/
│   │               │   └── User.java
│   │               └── repositories/
│   │                   └── UserRepository.java
│   └── resources/
└── test/
    ├── java/
    │   └── cothe/
    │       └── security/
    │           └── core/
    │               └── repositories/
    │                   └── UserRepositoryTest.java
    └── resources/
        └── application.yml
```

### 1) Entity Class 만들기

User Class 는 사용자의 기본 정보(id, password, 활성여부)을 관리할 클래스이다. `@Entity` 애노테이션으로 이 클래가 엔티티 클래스 임을 표시한다. `@Getter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@Builder` lombok 애노테이션으로 이 클래스에 대한 접근, 생성에 관한설정을 한다. lombok에 관한사항은 [여기](https://projectlombok.org/features/all)에서 자세히 알아보기 바란다.

`Users.java`

```java
package cothe.security.core.domain;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    @Id
    private String userId;
    private String password;
    private boolean enabled;
}
```

### 2) Repository Interface 만들기

JpaRepository interface를 상속받아 간단한 CRUD가 가능한 Repository를 만든다.

`UsersRepository.java`

```java
package cothe.security.core.repositories;

import cothe.security.core.domain.Users;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, String> {
}
```

### 3) Datasource 접속 정보 설정

Datasource 연결정보를 작성한다. 필요한 테이블이 있는 경우 자동으로 만들도록 설정했다.

`application.yml`

```yml
spring:
  profiles:
    active: local

# local 환경
---
spring:
  profiles: local
  datasource:
    url: jdbc:mysql://localhost:3306/security_db
    username: secuser
    password: sec1234
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create	
```
### 4) Jpa Test 작성

새 User 오브젝트를 만들어서 앞서 만든 UserRepository로 저장을 하고, 저장된 모든 사용자들 조회한 후 조금 전 새로 넣은 사용자 id 객체가 존재하는지 테스트한다.

`UserRepositoryTest.java`

```java
package cothe.security.core.repositories;

import cothe.security.core.domain.Users;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import java.util.List;
import static org.junit.Assert.assertTrue;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UsersRepositoryTest {
    @Autowired
    UsersRepository userRepository;

    @Test
    public void JPA로_MySql_접근() {
        //given
        userRepository.save(User.builder()
                .userId("testUser")
                .password("password")
                .enabled(true)
                .build());

        //when
        List<Users> users = userRepository.findAll();

        //then
        assertTrue(users.stream().anyMatch(user -> user.getUserId().equals("testUser")));
    }
}
```

### 5) 테스트 실행
테스트 작성한것을 실행해보면 녹색불이 잘 들어오고
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_135833.png)

로그에도 실제로 실행된 SQL이 잘 보인다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_135929.png)

### 6) jdbc Test 작성
이번에는 jdbcTemplate으로 직접 쿼리를 작성해서 db와 상호작용하는 테스트를 만든다.

`UserRepositoryTest.java`

```java
package cothe.security.core.repositories;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserRepositoryTest {
    @Autowired
    JdbcTemplate jdbcTemplate; // jdbcTemplate 을 주입받을 수 있도록 함

    ...

    @Test
    public void JDBC로_MySql_접근() {
        //given
        int cnt = jdbcTemplate.queryForObject("select count(*) from user where user_id = ?"
                , new Object[]{"jdbcTestUser"}, Integer.class);

        if(cnt == 0) {
            jdbcTemplate.update("insert into user(user_id,password,enabled) values (?,?,?)"
                    , "jdbcTestUser"
                    , "password"
                    , true);
        }

        //when
        List<User> users = jdbcTemplate.query("select * from user", (rs, rowNum) -> User.builder()
                .userId(rs.getString("user_id"))
                .password(rs.getString("password"))
                .enabled(rs.getBoolean("enabled"))
                .build()
        );

        //then
        assertTrue(users.stream().anyMatch(user -> user.getUserId().equals("jdbcTestUser")));
    }
}
```
### 7) 로깅 레벨 변경
실행된 쿼리가 로그에 남도록 `JdbcTemplate` class의 로그 레벨을 _debug_ 로 변경한다.

`application.yml`

```yml
spring:
  profiles:
    active: local

# local 환경
---
spring:
  profiles: local
  datasource:
    url: jdbc:mysql://localhost:3306/security_db
    username: secuser
    password: sec1234
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create    

logging:
  level.org.springframework.jdbc.core.JdbcTemplate: debug
```

### 8) 실행결과

기대했던 대로 녹색불이 들어온다.

![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_173429.png)

실행된 sql도 로그에 잘 남는 것을 볼 수 있다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_173519.png)

# Spring Security 인증 테스트
웹에서 일반적인 인증 절차는 로그인 폼에서 사용자 아이디와 비밀번호를 사용자로부터 입력받아 인증 서버로 전송하고 서버에서 그 요청에 있는 사용자 아이디와 비밀번호가 DB에 있는 것과 일치하면 인증에 성공한다. 이 절차를 테스트로 간단하게 만들어 보자.

스프링 시큐리티에서 인증은 `AuthenticationManager` 의 `authenticate` 메소드로 한다. 이 클래스의 대표적인 구현체인 `ProviderManager` 는 내부에 `AuthenticationProvider` 리스트를 가지고 있고 이 리스트에 있는 `AuthenticationProvider` 의 `authenticate` 메소드로 인증을 한다.

먼저 `ProviderManager` 객체를 만들어 보자. 생성자에 `AuthenticationProvider` 리스트가 필요함을 알 수 있다.
![](/assets/2018-07-31-spring-security-service/2018-07-31-spring-security-service_165016.png)

그럼 `ProviderManager`를 만들기 전에 `AuthenticationProvider` 리스트를 만들어 놔야 하는데 `AuthenticationProvider` 는 인터페이스이기 떄문에 구현하던지 스프링시큐리티에서 제공하는 구현체를 사용하던지 해야한다.
여기서는 `DaoAuthenticationProvider` 를 사용해보자. 이 클래스는 대표적인 사용자Id와 패스워드로 인증을 할 수 있도록 한 구현체이다.

`DaoAuthenticationProvider` 는 `UserDetails` 를 가져올 수 있는 `UserDetailsService` 와 `PasswordEncoder` 가 필요하다. `PasswordEncoder`는 일단 사용하지 않도록 `NoOpPasswordEncoder`를 넣어주고 `userDetailsService` 를 생각해보자.

```java
package cothe.security.authentication;

import org.junit.Test;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;

public class AuthenticateTest {
    @Test
    public void authenticate() {
        DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(???);
        authenticationProvider.setPasswordEncoder(NoOpPasswordEncoder.getInstance());

        ProviderManager providerManager = new ProviderManager(???)
    }
}
```

`UserDetailsService` 는 _username_ 으로 `UserDetails` 를 리턴하는 메소드 `loadUserByUsername()` 하나를 가진 인터페이스 이다. 
```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

이 클래스의 대표적인 구현체는 `InMemoryUserDetailsManager`, `JdbcUserDetailsManager` 등이 있다.

`UserDetails` 는 사용자 정보를 제공하는 인터페이스이다. 대표적인 구현체로 `User` 가 있다.

```java
package org.springframework.security.core.userdetails;

public interface UserDetails extends Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();
	String getPassword();
	String getUsername();
	boolean isAccountNonExpired();
	boolean isAccountNonLocked();
	boolean isCredentialsNonExpired();
	boolean isEnabled();
}
```

여기서는 `User` 객체를 리턴하는 `UserDetailsService` 인터페이스를 직접 구현하는 Mock Class를 구현한다.

```java
package cothe.security.core.userdetails;

public class MockUserDetailsService implements UserDetailsService {
    UserRepository userRepository;

    public MockUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<UserDetails> userDetails = userRepository.findById(username).map(user ->
                User.builder()
                        .username(user.getUserId())
                        .password(user.getPassword())
                        .authorities(Arrays.asList(new SimpleGrantedAuthority("Test")))
                        .build()
        );

        return userDetails.orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다."));
    }
}
```

여기서 `UserRepository`는 앞서 정의한 인터페이스인데 JPA 를 이용해서 직접 DB 에서 가져오는 것이 아니라 임의로 `User` 객체를 가지고 오도록 별도 구현 할 것이다. 또한 `loadUserByUsername` 메소드에서 `User` 객체를 만들때 `authorites` 는 임시로 하나만 하드코딩으로 넣어 두겠다. `loadUserByUsername` 은 `null`을 리턴하면 안된다. 만약 사용자를 찾을 수 없으면 `UsernameNotFoundException` 을 내도록 한다.

이제 `UserRepository` 를 구현하자. `UserRepository`는 메소드가 꽤 많은데 그중에 우리가 사용할 메도스 몇 개만 구현하도록 하겠다. 구현체 내부에 필드로 `HashMap` 을 가지고 새 사용자가 추가되면 여기에 저장했다가 사용자 요청을 받으면 여기에서 꺼내 리턴하도록 한다.

```java
package cothe.security.authentication;

public class MockUserRepository implements UserRepository {
    private Map<String, User> users = new HashMap<>();

    @Override
    public <S extends User> S save(S user) {
        this.users.put(user.getUserId(), user);
        return user;
    }

    @Override
    public Optional<User> findById(String userId) {
        return Optional.ofNullable(this.users.get(userId));
    }
}
```
이제 추가로 만든 Mock 클래스들과 합쳐서 테스트 코드를 정리하자.

```java
package cothe.security.authentication;

public class AuthenticateTest {
    @Test
    public void authenticate() {
        UserRepository userRepository = new MockUserRepository();
        userRepository.save(
                new User("cothe", "pass", true
                ));
        UserDetailsService userDetailsService = new MockUserDetailsService(userRepository);

        DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService);
        authenticationProvider.setPasswordEncoder(NoOpPasswordEncoder.getInstance());

        List<AuthenticationProvider> authenticationProviders =
                Stream.of(
                        authenticationProvider
                ).collect(Collectors.toList());

        ProviderManager providerManager = new ProviderManager(authenticationProviders);
    }
}
```

이제 인증할 준비는 다 됐고 사용자가 입력한 인증정보를 만들어 내자.

```java
Authentication auth = new UsernamePasswordAuthenticationToken("cothe", "pass");
```

`Authentication` 는 인증, 요청, 토큰 인증 상태들을 관리하는 인터페이스이다. 여기에서는 대표적인 구현체인 `UsernamePasswordAuthenticationToken` 를 이용해서 인증 토큰을 만들고 있다.
```java
package org.springframework.security.core;

public interface Authentication extends Principal, Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();
	Object getCredentials();
	Object getDetails();
	Object getPrincipal();
	boolean isAuthenticated();
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

지금까지 테스트 코드

```java
package cothe.security.authentication;

public class AuthenticateTest {
    @Test
    public void authenticate() {
        UserRepository userRepository = new MockUserRepository();
        userRepository.save(
                new User("cothe", "pass", true
                ));
        UserDetailsService userDetailsService = new MockUserDetailsService(userRepository);

        DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService);
        authenticationProvider.setPasswordEncoder(NoOpPasswordEncoder.getInstance());

        List<AuthenticationProvider> authenticationProviders =
                Stream.of(
                        authenticationProvider
                ).collect(Collectors.toList());

        ProviderManager providerManager = new ProviderManager(authenticationProviders);

        Authentication auth = new UsernamePasswordAuthenticationToken("cothe", "pass");
        
        assertFalse(auth.isAuthenticated());
        
        // 인증
        auth = providerManager.authenticate(auth);
        
        assertTrue(auth.isAuthenticated());
    }
}
```

만약 인증에 실패하게 된다면 옵션에 따라 발생하는 예외가 다르긴 한데 기본값으로 `BadCredentialsException` 이 발생하게 된다. 이 테스트 코드를 리펙토링해서 성공했을 때와 실패했을 때 테스트를 만들자.

```java
package cothe.security.authentication;

public class AuthenticateTest {

    private UserRepository userRepository;
    private UserDetailsService userDetailsService;
    private DaoAuthenticationProvider authenticationProvider;
    private List<AuthenticationProvider> authenticationProviders;
    private ProviderManager providerManager;

    @Before
    public void setUp() throws Exception {

        userRepository = new MockUserRepository();
        userRepository.save(
                new User("cothe", "pass", true
                ));

        userDetailsService = new MockUserDetailsService(userRepository);

        authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService);
        authenticationProvider.setPasswordEncoder(NoOpPasswordEncoder.getInstance());
        authenticationProviders = Stream.of(
                authenticationProvider
        ).collect(Collectors.toList());
        providerManager = new ProviderManager(authenticationProviders);
    }

    @Test
    public void 인증성공() {
        Authentication auth = new UsernamePasswordAuthenticationToken("cothe", "pass");

        assertFalse(auth.isAuthenticated());
        auth = providerManager.authenticate(auth);
        assertTrue(auth.isAuthenticated());
    }

    @Test(expected = BadCredentialsException.class)
    public void 인증실패(){
        Authentication auth = new UsernamePasswordAuthenticationToken("cothe1", "pass");
        auth = providerManager.authenticate(auth);
    }
}
```

![](/assets/2018-07-31-spring-security-service/2018-07-31-spring-security-service_181430.png)

다음 장에는 권한체크를 할 수 있도록 User 오브젝트에 Role을 넣는 것부터 시작할 것이다.
