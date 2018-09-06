---
layout: post
title: Spring Security로 Security 서비스 구축하기
description: Spring Security 보안 서비스 구축하기
date:   2018-07-16 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---

사용자관리, 권한설정, 접근권한 관리를 할 수 있는 서비스를 만들어 보자.
# 1. Data Architecture
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_164918.png)

```sql
-- 사용자
CREATE TABLE `security_db`.`USER` (
	`USER_ID`  VARCHAR(50)  NOT NULL COMMENT '사용자 ID', -- 사용자ID
	`PASSWORD` VARCHAR(200) NULL     COMMENT '패스워드', -- 패스워드
	`ENABLED`  BOOLEAN      NOT NULL COMMENT '가용' -- 가용
)
COMMENT '사용자';

-- 사용자
ALTER TABLE `security_db`.`USER`
	ADD CONSTRAINT `PK_USER` -- 사용자 기본키
		PRIMARY KEY (
			`USER_ID` -- 사용자ID
		);

-- 사용자ID별_역할
CREATE TABLE `security_db`.`USER_ROLE` (
	`USER_ID` VARCHAR(50) NOT NULL COMMENT '사용자ID', -- 사용자ID
	`ROLE_ID` VARCHAR(50) NOT NULL COMMENT 'ROLE ID' -- ROLE ID
)
COMMENT '사용자ID별_역할';

-- 사용자ID별_역할
ALTER TABLE `security_db`.`USER_ROLE`
	ADD CONSTRAINT `PK_USER_ROLE` -- 사용자ID별_역할 기본키
		PRIMARY KEY (
			`USER_ID`, -- 사용자ID
			`ROLE_ID`  -- ROLE ID
		);

-- 역할
CREATE TABLE `security_db`.`ROLE` (
	`ROLE_ID`        VARCHAR(50) NOT NULL COMMENT 'ROLE ID', -- ROLE ID
	`ROLE_NAME`      VARCHAR(50) NULL     COMMENT 'ROLE NAME', -- ROLE NAME
	`PARENT_ROLE_ID` VARCHAR(50) NULL     COMMENT '부모ROLE ID' -- 부모ROLE ID
)
COMMENT '역할';

-- 역할
ALTER TABLE `security_db`.`ROLE`
	ADD CONSTRAINT `PK_ROLE` -- 역할 기본키
		PRIMARY KEY (
			`ROLE_ID` -- ROLE ID
		);

-- ROLE_PERMISSION
CREATE TABLE `security_db`.`ROLE_PERMISSION` (
	`ROLE_ID`       VARCHAR(50) NOT NULL COMMENT 'ROLE ID', -- ROLE ID
	`PERMISSION_ID` VARCHAR(50) NOT NULL COMMENT '퍼미션 ID' -- 퍼미션 ID
)
COMMENT 'ROLE_PERMISSION';

-- ROLE_PERMISSION
ALTER TABLE `security_db`.`ROLE_PERMISSION`
	ADD CONSTRAINT `PK_ROLE_PERMISSION` -- ROLE_PERMISSION 기본키
		PRIMARY KEY (
			`ROLE_ID`,       -- ROLE ID
			`PERMISSION_ID`  -- 퍼미션 ID
		);

-- PERMISSION
CREATE TABLE `security_db`.`PERMISSION` (
	`PERMISSION_ID`   VARCHAR(50)   NOT NULL COMMENT '퍼미션 ID', -- 퍼미션 ID
	`OBJECT_ID`       VARCHAR(50)   NOT NULL COMMENT '오브젝트 ID', -- 오브젝트 ID
	`PERMISSION_NAME` VARCHAR(50)   NULL     COMMENT '퍼미션 명', -- 퍼미션 명
	`PERMISSION`      VARCHAR(4000) NULL     COMMENT '퍼미션 스크립트' -- 퍼미션 스크립트
)
COMMENT 'PERMISSION';

-- PERMISSION
ALTER TABLE `security_db`.`PERMISSION`
	ADD CONSTRAINT `PK_PERMISSION` -- PERMISSION 기본키
		PRIMARY KEY (
			`PERMISSION_ID` -- 퍼미션 ID
		);

-- OBJECT
CREATE TABLE `security_db`.`OBJECT` (
	`OBJECT_ID`        VARCHAR(50)  NOT NULL COMMENT '오브젝트 ID', -- 오브젝트 ID
	`OBJECT_NAME`      VARCHAR(50)  NULL     COMMENT '오브젝트 명', -- 오브젝트 명
	`OBJECT_TYPE`      VARCHAR(20)  NOT NULL COMMENT '오브젝트 유형', -- 오브젝트 유형
	`SYSTEM_CODE`      VARCHAR(20)  NOT NULL COMMENT '시스템 구분', -- 시스템 구분
	`SUB_SYSTEM_CODE`  VARCHAR(20)  NOT NULL COMMENT '체인코드', -- 체인코드
	`USE_YN`           VARCHAR(1)   NOT NULL COMMENT '사용여부', -- 사용여부
	`OBJECT_REFERENCE` VARCHAR(200) NULL     COMMENT '오브젝트 참조' -- 오브젝트 참조
)
COMMENT 'OBJECT';

-- OBJECT
ALTER TABLE `security_db`.`OBJECT`
	ADD CONSTRAINT `PK_OBJECT` -- OBJECT 기본키
		PRIMARY KEY (
			`OBJECT_ID` -- 오브젝트 ID
		);

-- 사용자ID별_역할
ALTER TABLE `security_db`.`USER_ROLE`
	ADD CONSTRAINT `FK_USER_TO_USER_ROLE` -- 사용자 -> 사용자ID별_역할
		FOREIGN KEY (
			`USER_ID` -- 사용자ID
		)
		REFERENCES `security_db`.`USER` ( -- 사용자
			`USER_ID` -- 사용자ID
		);

-- 사용자ID별_역할
ALTER TABLE `security_db`.`USER_ROLE`
	ADD CONSTRAINT `FK_ROLE_TO_USER_ROLE` -- 역할 -> 사용자ID별_역할
		FOREIGN KEY (
			`ROLE_ID` -- ROLE ID
		)
		REFERENCES `security_db`.`ROLE` ( -- 역할
			`ROLE_ID` -- ROLE ID
		);

-- 역할
ALTER TABLE `security_db`.`ROLE`
	ADD CONSTRAINT `FK_ROLE_TO_ROLE` -- 역할 -> 역할
		FOREIGN KEY (
			`PARENT_ROLE_ID` -- 부모ROLE ID
		)
		REFERENCES `security_db`.`ROLE` ( -- 역할
			`ROLE_ID` -- ROLE ID
		);

-- ROLE_PERMISSION
ALTER TABLE `security_db`.`ROLE_PERMISSION`
	ADD CONSTRAINT `FK_ROLE_TO_ROLE_PERMISSION` -- 역할 -> ROLE_PERMISSION
		FOREIGN KEY (
			`ROLE_ID` -- ROLE ID
		)
		REFERENCES `security_db`.`ROLE` ( -- 역할
			`ROLE_ID` -- ROLE ID
		);

-- ROLE_PERMISSION
ALTER TABLE `security_db`.`ROLE_PERMISSION`
	ADD CONSTRAINT `FK_PERMISSION_TO_ROLE_PERMISSION` -- PERMISSION -> ROLE_PERMISSION
		FOREIGN KEY (
			`PERMISSION_ID` -- 퍼미션 ID
		)
		REFERENCES `security_db`.`PERMISSION` ( -- PERMISSION
			`PERMISSION_ID` -- 퍼미션 ID
		);

-- PERMISSION
ALTER TABLE `security_db`.`PERMISSION`
	ADD CONSTRAINT `FK_OBJECT_TO_PERMISSION` -- OBJECT -> PERMISSION
		FOREIGN KEY (
			`OBJECT_ID` -- 오브젝트 ID
		)
		REFERENCES `security_db`.`OBJECT` ( -- OBJECT
			`OBJECT_ID` -- 오브젝트 ID
		);
```
# 2. 프로젝트 생성
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134112.png)
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134205.png)


Core에 Security와 Lombok을 선택한다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134231.png)

SQL에 JPA, MySql, JDBC를 선택한다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_134307.png)

그 뒤는 기본값으로 해서 프로젝트를 생성한다.

프로젝트가 생성됐으니 데이타베이스와 상호작용하는 코드를 간단하게 만들어보자.

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

`UsersRepository.java`

```java
package cothe.security.core.repositories;

import cothe.security.core.domain.Users;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

### 3) Datasource 접속 정보 설정

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
```
### 4) Jpa Test 작성

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

import cothe.security.core.domain.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;
import java.util.List;
import static org.junit.Assert.assertTrue;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserRepositoryTest {
    @Autowired
    UserRepository userRepository;

    @Autowired
    JdbcTemplate jdbcTemplate; // jdbcTemplate 을 주입받을 수 있도록 함

    @Test
    public void JPA로_MySql_접근() {
        //given
        userRepository.save(User.builder()
                .userId("testUser")
                .password("password")
                .enabled(true)
                .build());

        //when
        List<User> users = userRepository.findAll();

        //then
        assertTrue(users.stream().anyMatch(user -> user.getUserId().equals("testUser")));
    }

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

logging:
  level.org.springframework.jdbc.core.JdbcTemplate: debug
```

### 8) 실행결과
기대했던 대로 녹색불이 들어온다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_173429.png)

실행된 sql도 로그에 잘 남는 것을 볼 수 있다.
![](/assets/2018-07-16-spring-security-service/2018-07-16-spring-security-service_173519.png)





1.SecurityContextHolder에서 Authentication 개체를 가져옵니다.
2.SecurityMetadataSource에 대한 보안 오브젝트 요청을 조사하여 요청이 보안 호출 또는 공개 호출과 관련되는지 판별
3.보안 호출의 경우 (보안 오브젝트 호출에 대한 ConfigAttributes 목록이 있음)
3-1.Authentication.isAuthenticated ()가 false를 반환하거나 alwaysReauthenticate가 true이면 구성된 AuthenticationManager에 대해 요청을 인증
3-2.인증 될 때 SecurityContextHolder의 Authentication 개체를 반환 된 값으로 바꿉
3-3.구성된 RunAsManager를 통해 임의의 Run-as 대체를 수행
3-4.콘트롤을 구체적인 서브 클래스로 다시 넘겨 주면 실제로 객체를 수행하게됩니다.InterceptorStatusToken이 반환되어 서브 클래스가 객체의 실행을 끝내고 나서 finally 절을 사용하면 finallyInvocation (InterceptorStatusToken)을 사용하여 AbstractSecurityInterceptor를 다시 호출하고 올바르게 정리할 수 있습니다.
3-5.구체적인 서브 클래스는 afterInvocation (InterceptorStatusToken, Object) 메소드를 통해 AbstractSecurityInterceptor를 다시 호출합니다.
3-6.RunAsManager가 Authentication 개체를 바꾼 경우 SecurityManager를 호출 한 후에 있던 개체에 SecurityContextHolder를 반환합니다.
3-7.AfterInvocationManager가 정의 된 경우 호출 관리자에게 호출하고 호출자에게 리턴 될 예정으로 인해 오브젝트를 대체하도록 허용하십시오.
4.공용 인 호출의 경우 (보안 오브젝트 호출에 대한 ConfigAttributes가 없음).
4-1.전술 한 바와 같이, 구체적인 서브 클래스는 InterceptorStatusToken으로 리턴 될 것이고, 보안 객체가 실행 된 후에 AbstractSecurityInterceptor로 다시 제시된다. AbstractSecurityInterceptor는 afterInvocation (InterceptorStatusToken, Object)이 호출 될 때 더 이상의 조치를 취하지 않습니다.
5.컨트롤은 다시 호출자에게 반환되어야하는 Object와 함께 구체적인 하위 클래스로 반환됩니다. 서브 클래스는 그 결과 나 예외를 원 호출자에게 리턴 할 것이다.


AffirmativeBased의 부모 클래스인 AbstractAccessDecisionManager 의 supports는 AccessDecisionManager가 가지고 있는 Voter들의 supports를 하나씩 돌려봐서 하나라도 true가 리턴되면 전체 true로 리턴함



Web Security
DelegatingFilterProxy 라는 Filter 를 web.xml 에 등록시킨다.
DelegatingFilterProxy 는 자체적으로 비즈니스 로직을 가지지 않고 스프링 애플리케이션 컨텍스트에서 가져온 필터들의 메소드를 대신 실행시켜주는 역할을 한다. 스프링 애플리케이션 컨텍스트에 등록된 빈들은 javax.servlet.Filter 를 구현해야 한다.
스프링 시큐리티의 웹 인프라는 FilterChainProxy 인스턴스에 위임해야만 사용할 수 있다. 스큐리티 필터들을 독립적으로 사용하면 안된다. 이론적으로는 web.xml 에 등록해서 쓸 수 있지만 번거롭고 복잡하므로 그렇게 하지 마라.
