---
layout: post
title: Spring Security 6에서 인증 정보가 세션에 저장되지 않는 문제
description: Spring Security 6에서 인증 정보가 세션에 저장되지 않는 문제
date:   2026-01-29 16:50:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	Spring Security Security Context
---
# Spring Security 6에서 인증 정보가 세션에 저장되지 않는 문제

API에서 직접 로그인 로직을 구현했는데, 인증은 성공하지만 다음 요청에서 인증 정보가 사라지는 경험을 해보셨나요? 이번 포스트에서는 이 문제의 원인과 해결 방법을 정리해보겠습니다.

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


## 문제 상황

API에서 직접 로그인 로직을 구현했고, 인증도 성공했습니다. 그런데 다음 요청에서 인증 정보가 사라져버렸습니다.

콘솔에는 다음과 같은 로그가 찍힙니다.

```
Did not find SecurityContext in HttpSession ... using the SPRING_SECURITY_CONTEXT session attribute
```

즉, HttpSession 안에서 SecurityContext를 찾지 못한다는 의미입니다.

## 원인: Spring Security 6의 기본 동작 변경

Spring Security 6(Spring Boot 3.x)부터는 SecurityContext 자동 저장이 기본적으로 꺼져 있습니다.

정확히 말하면 `requireExplicitSave` 기본값이 활성화되어, 개발자가 명시적으로 저장하지 않으면 세션에 남지 않습니다. 이전 버전처럼 `SecurityContextHolder`에만 넣으면 세션에는 저장되지 않습니다.

이는 Spring Security의 보안 및 성능 개선을 위한 의도적인 변경 사항입니다.

## formLogin은 왜 문제없을까?

그런데 `formLogin`을 사용하면 이런 문제가 발생하지 않습니다. 왜 그럴까요?

`formLogin`을 사용하면 내부 필터들이 다음 단계를 모두 수행하기 때문입니다.

1. `UsernamePasswordAuthenticationFilter`가 인증 처리
2. 성공 시 SecurityContext 생성
3. `SecurityContextPersistenceFilter`가 HttpSession에 저장

즉, `formLogin`은 인증 이후 세션 저장까지 프레임워크가 자동으로 처리해줍니다.

## 수동 로그인에서 해결 방법

직접 로그인 로직을 구현한 경우에는 명시적 저장이 필요합니다.

핵심은 다음 한 줄입니다.

```java
securityContextRepository.saveContext(context, request, response);
```

이걸 호출해야 `SPRING_SECURITY_CONTEXT`가 세션에 들어갑니다.

### 전체 코드 예시

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final SecurityContextRepository securityContextRepository;

    public AuthController(
            AuthenticationManager authenticationManager,
            SecurityContextRepository securityContextRepository) {
        this.authenticationManager = authenticationManager;
        this.securityContextRepository = securityContextRepository;
    }

    @PostMapping("/login")
    public ResponseEntity<?> login(
            @RequestBody LoginRequest loginRequest,
            HttpServletRequest request,
            HttpServletResponse response) {
        
        // 1. 인증 수행
        UsernamePasswordAuthenticationToken token = 
            new UsernamePasswordAuthenticationToken(
                loginRequest.getUsername(), 
                loginRequest.getPassword()
            );
        
        Authentication authentication = authenticationManager.authenticate(token);
        
        // 2. SecurityContext 생성 및 설정
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);
        SecurityContextHolder.setContext(context);
        
        // 3. 세션에 명시적으로 저장 (핵심!)
        securityContextRepository.saveContext(context, request, response);
        
        return ResponseEntity.ok("로그인 성공");
    }
}
```

### SecurityContextRepository 빈 설정

`SecurityContextRepository`를 주입받으려면 다음과 같이 빈으로 등록해야 합니다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityContextRepository securityContextRepository() {
        return new HttpSessionSecurityContextRepository();
    }

    // ... 나머지 Security 설정
}
```

## 정리

핵심 내용을 정리하면 다음과 같습니다.

- Spring Security 6부터는 SecurityContext가 자동으로 세션에 저장되지 않습니다
- `formLogin`은 내부 필터에서 자동 저장하지만, 수동 로그인은 직접 저장해야 합니다
- 해결은 `SecurityContextRepository.saveContext(...)` 호출입니다

## 추가 체크 포인트

인증 정보가 세션에 제대로 저장되더라도 다음 사항들을 확인해야 합니다.

### 1. 쿠키 전송 확인

클라이언트는 반드시 `JSESSIONID` 쿠키를 다음 요청에 포함해야 합니다. 

브라우저는 기본적으로 쿠키를 자동으로 포함하지만, Postman이나 axios 같은 클라이언트를 사용할 때는 명시적으로 쿠키 전송을 설정해야 합니다.

```javascript
// axios 예시
axios.post('/api/auth/login', credentials, {
    withCredentials: true  // 쿠키 포함
});
```

### 2. 세션 저장소

기본 세션 저장소는 서버 메모리(톰캣 세션)입니다. 

이는 다음과 같은 한계가 있습니다.

- 서버 재시작 시 세션 소멸
- 다중 서버 환경에서 세션 공유 불가

### 3. 분산 환경 고려

분산 환경이라면 Spring Session + Redis 같은 외부 세션 저장소를 고려해야 합니다.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  session:
    store-type: redis
```

이렇게 하면 여러 서버 인스턴스 간에 세션을 공유할 수 있습니다.

## 마무리

Spring Security 6의 변경사항은 명시적 보안 관리를 권장하는 방향입니다. 처음에는 불편할 수 있지만, 이를 통해 개발자가 세션 관리를 더 명확하게 제어할 수 있게 되었습니다.

수동 로그인을 구현할 때는 반드시 `SecurityContextRepository.saveContext()`를 호출하는 것을 잊지 마세요!


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