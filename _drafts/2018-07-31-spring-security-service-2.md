---
layout: post
title: Spring Security로 Security 서비스 구축하기 1
description: Spring Security로 인증
date:   2018-07-31 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---


# 사용자 오브젝트가 권한을 가지도록 변경

`User` Class에 `roles` 필드 추가
```java
package cothe.security.core.domain;

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
    private Set<Role> roles;
}
```
Role Class 추가

```java
package cothe.security.core.domain;

@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Role {
    private String roleId;
    private String roleName;
    private Role parentRole;
}
```

인증테스트의 `setUp` 부분에서 `Role` 오브젝트를 저장하도록 수정

`AuthenticateTest.java`

```java
    ...

    @Before
    public void setUp() throws Exception {

        userRepository = new MockUserRepository();
        userRepository.save(
                new User("cothe", "pass", true,
                        Stream.of(Role.builder()
                                .roleId("admin")
                                .roleName("관리자").parentRole(null)
                                .build()).collect(Collectors.toSet())
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

    ...
```

# Role을 Entity 클래스로 설정

### Role을 Entity로

```java
package cothe.security.core.domain;

@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Role {
    @Id
    private String roleId;
    private String roleName;

    @ManyToOne()
    @JoinColumn(name = "parent_user_id")
    private Role parentRole;
}
```

### User와 Role의 관계 설정

```java
package cothe.security.core.domain;

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

    @OneToMany
    private Set<Role> roles;
}
```
### 테스트
테스트가 정상적으로 통과됐고 DB에는 테이블 3개가 생성된 것을 볼 수 있다.
![](/assets/2018-07-31-spring-security-service/2018-07-31-spring-security-service_124824.png)

# Spring Security 접근 권한 테스트
앞선 실습에서 `User` 도메인 객체가 `Role` 들을 가지고 있도록 했다. 이 객체로 인증을 하기 위해서 Spring Security의 `User` 클래스로 랩핑을 했는데 그 과정에서 `Set<Role>` 을 `Set<SimpleGrantedAuthority>` 로 전환하는 과정이 있었다. 이 정보를 `AccessDecisionManager` 가 읽어서 권한 체크를 한다. 스프링 시큐리티에서 접근 권한체크를 하는 일련의 과정을 실습을 해보겠다.  

 먼저 스프링 시큐리티의 접근 권한 관련 클래스들의 구조를 보자.

![](/assets/2018-07-31-spring-security-service/2018-07-31-spring-security-service_170744.png)

최상단에 `AccessDecisionManager` 가 있다. 이 인터페이스는 `decide` 메소드로 권한 체크를 하게 되는데 권한이 없으면 `AccessException` 을 던지도록 되어있다.

```java
package org.springframework.security.access;

public interface AccessDecisionManager {
	void decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
			InsufficientAuthenticationException;
	boolean supports(ConfigAttribute attribute);
	boolean supports(Class<?> clazz);
}
```

`AccessDecisionManager` 를 구현한 추상클래스 `AbstractAccessDecisionManager` 는 내부에 `AccessDecisionVoter` 인터페이스 리스트를 저장할 수 있도록 필드가 있다. 이 리스트는 `AccessDecisionManager` 가 생성될 때 초기화되어야 한다. 
`AccessDecisionVoter` 인터페이스는 접근 권한을 실제로 확인하는 vote 메소드를 가지고 있는데 권한체크 후 `int` 형으로 결과를 돌려준다.

리턴값 | 의미 
---------|----------
 1 | 권한 있음
 0 | 기권
 -1 | 권한 없음


이 `AccessDecisionVoter` 리스트를 어떤 전략으로 체크해서 최종 접근 권한 결정을 하는가에 따라서 구현된 3개 클래스가 `AffirmativeBased`, `ConsensusBased`, `UnanimousBased` 이다. 

 `AffirmativeBased` 는 `AccessDecisionVoter` 중 하나라도 1(권한있음) 이 리턴되면 접근 권한이 있다고 결정한다. `Voter`가 모두 기권인 경우는 별도 설정값에 따라 결정한다.  
 `ConsensusBased` 는 `AccessDecisionVoter` 들이 리턴한 값을 모두 더해서 양수인지 음수인지에 따라서 결정하고 합이 0인 경우와 `AccessDecisionVoter` 들이 모두 기권한 경우는 프로퍼티 설정값에 따라 결정한다.  
 `UnanimousBased` 는 하나라도 -1(권한없음)이 리턴되면 접근 권한이 없다고 결정한다.
 
 