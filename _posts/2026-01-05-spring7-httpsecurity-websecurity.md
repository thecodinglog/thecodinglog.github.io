---
layout: post
title: Spring Security 7.x HttpSecurity와 WebSecurity의 차이점
description: Spring Security 7.x HttpSecurity와 WebSecurity의 차이점 완벽 가이드
date:   2026-01-05 13:00:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	Spring Security HttpSecurity WebSecurity
---

Spring Security 7.x는 단순한 버전 업그레이드가 아닌, 보안 설정 패러다임의 근본적인 전환을 의미합니다. 특히 `HttpSecurity`와 `WebSecurity`의 역할과 사용 방식이 명확히 재정의되면서, 이를 제대로 이해하지 못하면 보안 헤더 누락, 필터 미적용, 예상치 못한 인증 우회 등의 문제가 발생할 수 있습니다.

이 글에서는 Spring Security 6.4.5와 7.0.2의 차이점을 중심으로, 실무에서 반드시 알아야 할 핵심 개념과 실전 팁을 다룹니다.

---

## 목차

1. Spring Security 7.0.2의 핵심 변화 요약
2. HttpSecurity와 WebSecurity 소개 및 역할 정의
3. HttpSecurity 상세 분석 및 실전 예제
4. WebSecurity 상세 분석 - 언제, 왜 사용하지 말아야 하는가
5. 두 구성 요소의 주요 차이점 (6.x vs 7.x)
6. 실제 애플리케이션에서의 활용 방법
7. 결론 및 마이그레이션 가이드

---

## 1. Spring Security 7.0.2의 핵심 변화 요약

### 1-1. Spring Security 7.0.2 개요

Spring Security 7.x는 6.x의 과도기적 요소들을 완전히 제거하고, **명시성(Explicitness)**과 **안전성(Safety)**을 최우선 가치로 삼은 완결판입니다. 이는 단순히 API가 바뀐 것이 아니라, "보안 설정은 암묵적이면 안 된다"는 철학의 완전한 구현입니다.

Spring Security 7.0.2의 주요 변경사항:

- **Lambda DSL의 강제화**: 모든 보안 구성은 람다 기반으로만 작성 가능
- **`.and()` 메서드 완전 제거**: 체이닝 기반 DSL의 종료
- **WebSecurityConfigurerAdapter 제거 유지**: 컴포넌트 기반 구성(Bean 기반)이 유일한 표준으로 완전히 정착
- **Virtual Thread 최적화**: `SecurityContextHolderStrategy` 개선
- **AuthorizationManager 완전 정착**: 모든 권한 처리의 표준화
- **보안 헤더 기본값 강화**: CSP, HSTS 등의 엄격한 기본 정책

---

### 1-2. 6.x 대비 7.x의 철학적 변화

| 항목 | Spring Security 6.x | Spring Security 7.x |
|------|---------------------|---------------------|
| **설계 철학** | 유연성 중시 (Flexibility) | 명시성 중시 (Explicitness) |
| **DSL 방식** | Lambda 권장, 체이닝 허용 | **Lambda 강제** |
| **`.and()` 메서드** | 사용 가능 | **완전 제거** |
| **WebSecurity 사용** | 종종 사용 | **극소수 케이스만** |
| **기본 보안 정책** | 관대한 기본값 | **엄격한 기본값** |
| **실수 허용도** | 높음 (경고 수준) | **낮음 (Fail Fast)** |

> 🔒 **왜 이렇게 바뀌었는가?**  
> 보안 취약점 리포트 분석 결과, 대부분의 문제가 "설정 누락" 또는 "암묵적 기본값에 대한 오해"에서 발생했습니다. 7.x는 이를 근본적으로 방지하기 위해 설계되었습니다.

---

### 1-3. "필터를 태우지 않는 설정"에 대한 강경한 입장

Spring Security 7.x에서 `WebSecurity.ignoring()`은 더 이상 "편리한 옵션"이 아닙니다.

**ignoring() 사용 시 발생하는 문제:**

- Spring Security 필터 체인 완전 우회
- `SecurityContextHolder` 비활성화
- 모든 보안 헤더 제거 (CSP, HSTS, X-Frame-Options 등)
- CSRF / CORS 완전 무시
- 감사 로그 미생성

➡️ **7.x에서는 사실상 "최후의 탈출구" 수준으로 격하**

---

### 1-4. Virtual Thread 시대에 맞춘 내부 구조 정비

Spring Security 7.x는 Java 21의 Virtual Thread와의 호환성을 고려하여 내부 구조가 개선되었습니다:

- `SecurityContextHolderStrategy`의 Virtual Thread 친화적 개선
- ThreadLocal에서 Scoped Context로의 전환 준비
- 비동기 필터 체인에서의 컨텍스트 유실 위험 감소

> ⚠️ **주의사항**  
> 사용자 정의 ThreadLocal 직접 접근은 여전히 위험합니다. `SecurityContextHolder.getContext()` 외의 직접 참조는 금지됩니다.

---

## 2. HttpSecurity와 WebSecurity 소개 및 역할 정의

### 2-1. HttpSecurity: "보안 정책을 정의하는 곳"

`HttpSecurity`는 Spring Security 7.x에서 **유일하고 중심적인 보안 구성 도구**입니다.

**HttpSecurity가 답하는 질문:**
> "이 HTTP 요청은 어떤 보안 규칙을 거쳐야 하는가?"

**담당 영역:**

- **인증(Authentication)**: 로그인 방식, 인증 제공자 설정
- **인가(Authorization)**: URL 패턴별 접근 권한
- **CSRF 보호**: Cross-Site Request Forgery 방어
- **CORS 설정**: Cross-Origin Resource Sharing 정책
- **세션 관리**: 세션 생성 정책, 동시 세션 제어
- **보안 헤더**: CSP, HSTS, X-Frame-Options 등
- **로그인/로그아웃**: 폼 로그인, OAuth2, 로그아웃 처리

**결과물:**  
`SecurityFilterChain` 빈 - Spring Security의 핵심 필터 체인

---

### 2-2. WebSecurity: "아예 보안을 적용할 것인가?"

`WebSecurity`는 Spring Security 7.x에서 **거의 사용하지 않아야 할 저수준 API**입니다.

**WebSecurity가 답하는 질문:**
> "이 요청을 Spring Security 세계로 들일 것인가?"

**담당 영역:**

- `FilterChainProxy` 생성 (내부적)
- 완전한 필터 우회 결정 (극히 제한적)

**7.x의 권장사항:**  
거의 모든 경우에 `HttpSecurity`의 `permitAll()`을 사용하고, `WebSecurity.ignoring()`은 피해야 합니다.

---

### 2-3. 근본적 역할 차이의 이해

```
요청 흐름:

1. Servlet Container
   ↓
2. WebSecurity (FilterChainProxy 생성)
   ↓
3. HttpSecurity (SecurityFilterChain)
   ↓ 
4. Application
```

- **WebSecurity**: "이 요청을 Spring Security가 볼 것인가?"
- **HttpSecurity**: "이 요청에 어떤 보안 규칙을 적용할 것인가?"

---

## 3. HttpSecurity 상세 분석 및 실전 예제

### 3-1. securityMatcher() vs requestMatchers() - 가장 많이 혼동하는 개념

Spring Security 7.0.2에서 이 두 메서드의 차이를 명확히 이해하는 것이 핵심입니다.

#### securityMatcher()
- **목적**: 특정 `SecurityFilterChain` 빈이 처리할 URL 패턴 지정
- **적용 계층**: 필터 체인 수준 (FilterChain Selection)
- **의미**: "이 필터 체인을 탈지 말지 결정"
- **우선순위**: `@Order` 어노테이션으로 결정

#### requestMatchers()
- **목적**: 선택된 `SecurityFilterChain` 내에서 URL별 보안 규칙 정의
- **적용 계층**: 인가 규칙 수준 (Authorization Rules)
- **의미**: "체인 안에서의 세부 권한 설정"
- **우선순위**: 먼저 정의된 패턴이 우선 (구체적 → 일반적 순서)

**비유로 이해하기:**

```
securityMatcher()   = 건물의 입구 (어느 건물로 들어갈지)
requestMatchers()   = 건물 내부의 층별 접근 권한 (몇 층까지 갈 수 있는지)
```

---

### 3-2. Spring Security 7.0.2의 HttpSecurity 구성 예제

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // 메서드 보안 활성화를 위한 표준 어노테이션
public class SecurityConfig {

    /**
     * API 전용 SecurityFilterChain
     * - Stateless 세션 정책
     * - JWT 기반 인증
     * - CSRF 비활성화
     */
    @Bean
    @Order(1)  // 우선순위 높음
    public SecurityFilterChain apiSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            // securityMatcher: 이 체인이 /api/**에만 적용됨을 명시
            .securityMatcher("/api/**")
            
            // requestMatchers: 체인 내부에서의 세부 규칙
            // securityMatcher에서 /api 로 시작하기 때문에
            // authorizeHttpRequests도 모두 /api로 시작해야함
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasAuthority("ADMIN")
                .anyRequest().authenticated()
            )
            
            // Stateless 세션 - JWT 환경
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            
            // CSRF 비활성화 (API는 토큰 기반)
            .csrf(csrf -> csrf.disable())
            
            // HTTP Basic 인증
            .httpBasic(withDefaults())
            
            // OAuth2 Resource Server 설정
            .oauth2ResourceServer(oauth2 -> 
                oauth2.jwt(withDefaults())
            );

        return http.build();
    }

    /**
     * 웹 애플리케이션용 SecurityFilterChain
     * - 세션 기반 인증
     * - 폼 로그인
     * - CSRF 활성화
     */
    @Bean
    @Order(2)  // API 체인 다음 순위
    public SecurityFilterChain webSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            // securityMatcher: 모든 요청 (API 제외)
            .securityMatcher("/**")
            
            .authorizeHttpRequests(authorize -> authorize
                // 정적 리소스 - ignoring() 대신 permitAll() 사용
                .requestMatchers(
                    "/resources/**", 
                    "/static/**", 
                    "/css/**", 
                    "/js/**", 
                    "/images/**",
                    "/webjars/**",
                    "/favicon.ico"
                ).permitAll()
                
                // 공개 페이지
                .requestMatchers("/", "/home", "/register", "/about").permitAll()
                
                // Actuator 엔드포인트
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                
                // Swagger 문서
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                
                // 관리자 전용
                .requestMatchers("/admin/**").hasAuthority("ADMIN")
                
                // 일반 사용자
                .requestMatchers("/user/**").hasAuthority("USER")
                
                // 나머지는 인증 필요
                .anyRequest().authenticated()
            )
            
            // 폼 로그인 설정
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/login")
                .defaultSuccessUrl("/dashboard")
                .failureUrl("/login?error")
                .permitAll()
            )
            
            // 로그아웃 설정
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout")
                .deleteCookies("JSESSIONID")
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .permitAll()
            )
            
            // CSRF 설정 (기본 활성화)
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            )
            
            // 세션 관리
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .maximumSessions(1)
                .maxSessionsPreventsLogin(false)
                .expiredUrl("/login?expired")
            )
            
            // 보안 헤더
            .headers(headers -> headers
                .frameOptions(frame -> frame.sameOrigin())
                .contentSecurityPolicy(csp -> 
                    csp.policyDirectives("script-src 'self'; style-src 'self'")
                )
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000)
                )
            )
            
            // Remember Me
            .rememberMe(remember -> remember
                .key("uniqueAndSecretKey")
                .tokenValiditySeconds(86400)  // 24시간
            );

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean 
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder().encode("password"))
            .authorities("USER")
            .build();
            
        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder().encode("admin"))
            .authorities("ADMIN", "USER")
            .build();
            
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

---

### 3-3. 7.x에서 의미가 달라진 설정들

Spring Security 7.x에서는 일부 설정의 의미와 효과가 이전 버전과 다릅니다.

| 설정 | 효과 | 사용 사례 |
|------|------|----------|
| `securityContext().disable()` | SecurityContext 생성/저장 완전 비활성화 | **완전 Stateless API** (JWT만 사용) |
| `requestCache().disable()` | 인증 후 원래 요청 복원 기능 제거 | 로그인 리다이렉트 불필요한 API |
| `sessionManagement().disable()` | 세션 관리 필터 자체 제거 | Stateless 전용 체인 |
| `csrf().disable()` | CSRF 보호 완전 비활성화 | RESTful API (주의 필요) |

> ⚠️ **중요**  
> 이러한 설정들은 정적 리소스나 API 전용 체인에서 성능 최적화를 위해 사용되지만, 잘못 적용하면 보안 취약점이 될 수 있습니다.

---

### 3-4. 정적 리소스 처리 - 7.x의 올바른 방법

**❌ 잘못된 방법 (WebSecurity.ignoring())**

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return (web) -> web
        .ignoring()
        .requestMatchers("/css/**", "/js/**");
}
```

**문제점:**
- 모든 보안 헤더 미적용 (CSP, HSTS 등)
- XSS 공격 위험 증가
- HTTPS 강제 불가
- 감사 로그 미생성

---

**✅ 올바른 방법 (HttpSecurity.permitAll())**

```java
@Bean
@Order(0)  // 최우선 처리
public SecurityFilterChain staticResourceChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/css/**", "/js/**", "/images/**", "/resources/**")
        .authorizeHttpRequests(auth -> 
            auth.anyRequest().permitAll()
        )
        // 성능 최적화를 위한 불필요한 필터 제거
        .securityContext(context -> context.disable())
        .requestCache(cache -> cache.disable())
        .sessionManagement(session -> session.disable());
    
    return http.build();
}
```

**장점:**
✔ 보안 헤더 유지 (CSP, HSTS, X-Frame-Options)  
✔ 필터 순서 보장  
✔ 감사 로그 생성 가능  
✔ HTTPS 강제 가능  
✔ 성능 최적화 (불필요한 필터만 제거)

---

## 4. WebSecurity 상세 분석 - 언제, 왜 사용하지 말아야 하는가

### 4-1. Spring Security 7.0.2에서의 WebSecurity

Spring Security 7.x에서 `WebSecurity`의 사용 범위는 극도로 제한되었습니다.

**WebSecurity를 여전히 사용할 수 있는 경우:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    /**
     * 극히 예외적인 경우에만 사용
     * - Servlet Container 레벨에서 처리되는 경로
     * - Spring Security가 기술적으로 처리 불가능한 요청
     */
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web
            .ignoring()
            .requestMatchers("/actuator/health", "/actuator/info");
    }
}
```

---

### 4-2. WebSecurity.ignoring()이 위험한 이유

**실제 보안 사고 시나리오:**

#### 시나리오 1: XSS 공격 노출

```java
// 잘못된 설정
web.ignoring().requestMatchers("/css/**");
```

**문제:**
- CSP(Content Security Policy) 헤더 미적용
- 악의적인 JavaScript가 CSS 파일에 삽입되어도 브라우저가 차단하지 않음
- XSS 공격 성공률 증가

---

#### 시나리오 2: HTTPS 강제 실패

```java
// 잘못된 설정
web.ignoring().requestMatchers("/images/**");
```

**문제:**
- HSTS 헤더 미적용
- 이미지 요청이 HTTP로 다운그레이드 될 수 있음
- 중간자 공격(MITM) 가능

---

#### 시나리오 3: 감사 로그 누락

```java
// 잘못된 설정
web.ignoring().requestMatchers("/api/public/**");
```

**문제:**
- `SecurityContextHolder` 비활성화
- 누가, 언제, 어떤 리소스에 접근했는지 추적 불가
- 컴플라이언스 감사 실패

---

### 4-3. WebSecurity 사용이 허용되는 유일한 경우

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return (web) -> web
        .ignoring()
        .requestMatchers(
            // Actuator의 일부 health check 엔드포인트
            // (로드밸런서가 매 초마다 호출하는 경우)
            "/actuator/health/liveness",
            "/actuator/health/readiness"
        );
}
```

**허용 조건:**
1. 초당 수천 건 이상의 요청이 발생하는 헬스체크
2. 보안 헤더가 기술적으로 불필요한 경로
3. 다른 방법으로는 성능 문제 해결이 불가능한 경우

> 📌 **보안팀 기준**  
> "WebSecurity.ignoring()은 곧 보안 책임 포기를 의미합니다. 사용 전 보안 검토가 필수입니다."

---

## 5. 두 구성 요소의 주요 차이점 (6.x vs 7.x)

### 5-1. 개념적 차이점

| 구분 | HttpSecurity | WebSecurity |
|------|--------------|-------------|
| **역할** | HTTP 요청에 대한 보안 정책 정의 | 전역 웹 보안 및 완전 우회 결정 |
| **주요 기능** | 인증, 인가, CSRF, 세션, 헤더 | FilterChainProxy 생성, ignoring() |
| **필터 적용** | 보안 필터 체인 통과 | **필터 체인 완전 우회** |
| **보안 헤더** | 적용됨 | **적용 안 됨** |
| **감사 로그** | 생성 가능 | **생성 불가** |
| **7.x 사용 빈도** | **높음 (거의 모든 설정)** | **극히 낮음** |

---

### 5-2. 버전별 변화 추이

| 항목 | Spring Security 6.4.5 | Spring Security 7.0.2 |
|------|----------------------|----------------------|
| **정적 리소스 처리** | WebSecurity.ignoring() 사용 가능 | **HttpSecurity.permitAll() 강력 권장** |
| **DSL 방식** | Lambda 권장, 체이닝 허용 | **Lambda 강제** |
| **`.and()` 메서드** | 사용 가능 (Deprecated) | **완전 제거** |
| **기본 보안 정책** | 관대한 기본값 | **엄격한 기본값** |
| **WebSecurity 철학** | "편의성 제공" | **"최후의 탈출구"** |
| **설정 누락 시** | 경고 | **Fail Fast (예외 발생)** |

---

### 5-3. securityMatcher vs requestMatcher 비교 (7.x 중점)

Spring Security 7.0.2에서 가장 중요한 개념 구분:

```java
http
    // ① securityMatcher: FilterChain 레벨 (어느 체인으로 들어갈까?)
    .securityMatcher("/api/**")
    
    .authorizeHttpRequests(authorize -> authorize
        // ② requestMatchers: Authorization 레벨 (체인 내부 규칙)
        .requestMatchers("/api/public/**").permitAll()
        .requestMatchers("/api/admin/**").hasAuthority("ADMIN")
        .anyRequest().authenticated()
    );
```

| 구분 | securityMatcher() | requestMatchers() |
|------|------------------|-------------------|
| **계층** | FilterChain Selection | Authorization Rules |
| **목적** | "이 체인을 사용할지 결정" | "체인 내부의 인가 규칙" |
| **우선순위** | @Order로 결정 | 선언 순서 (구체적 → 일반적) |
| **사용 위치** | SecurityFilterChain 최상단 | authorizeHttpRequests() 내부 |
| **예시** | `/api/**`, `/admin/**` | `/api/public/**`, `/admin/users/**` |

**실수 예방 팁:**
```java
// ❌ 잘못된 사용
http
    .securityMatcher("/api/**")
    .authorizeHttpRequests(auth -> 
        // securityMatcher가 /api/**인데 /admin/** 규칙?
        auth.requestMatchers("/admin/**").hasRole("ADMIN")
    );

// ✅ 올바른 사용
http
    .securityMatcher("/api/**")
    .authorizeHttpRequests(auth -> 
        auth.requestMatchers("/api/admin/**").hasRole("ADMIN")  // 일관성
    );
```

---

### 5-4. 필터 순서와 우선순위

```
요청: /api/admin/users

1단계: SecurityFilterChain 선택 (securityMatcher)
   - @Order(1): securityMatcher("/api/**") → 매칭! 이 체인 사용
   - @Order(2): securityMatcher("/**") → 평가 안 함

2단계: Authorization 규칙 적용 (requestMatchers)
   - requestMatchers("/api/public/**") → 불일치
   - requestMatchers("/api/admin/**") → 매칭! hasAuthority("ADMIN") 적용
```

---

## 6. 실제 애플리케이션에서의 활용 방법

### 6-1. 다중 SecurityFilterChain 설계 패턴

Spring Security 7.0.2에서는 **"체인은 많아져도 된다"**는 것이 핵심 원칙입니다.

**권장 설계 패턴:**

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class MultiChainSecurityConfig {

    /**
     * 1. 정적 리소스 체인 (최우선)
     * - 성능 최적화
     * - 보안 헤더는 유지
     */
    @Bean
    @Order(1)
    public SecurityFilterChain staticResourceChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/css/**", "/js/**", "/images/**", "/webjars/**")
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
            .securityContext(ctx -> ctx.disable())
            .requestCache(cache -> cache.disable())
            .sessionManagement(session -> session.disable());
        
        return http.build();
    }

    /**
     * 2. 공개 API 체인
     * - 인증 불필요
     * - Rate Limiting 적용
     */
    @Bean
    @Order(2)
    public SecurityFilterChain publicApiChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/public/**")
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .csrf(csrf -> csrf.disable());
        
        return http.build();
    }

    /**
     * 3. 인증 필요 API 체인
     * - JWT 기반 인증
     * - Stateless
     */
    @Bean
    @Order(3)
    public SecurityFilterChain authenticatedApiChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/admin/**").hasAuthority("ADMIN")
                .requestMatchers("/api/user/**").hasAuthority("USER")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .csrf(csrf -> csrf.disable())
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(withDefaults()));

        return http.build();
    }

    /**
     * 4. 관리자 페이지 체인
     * - 폼 로그인
     * - 세션 기반
     * - 추가 보안 검증
     */
    @Bean
    @Order(4)
    public SecurityFilterChain adminWebChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/admin/**")
            .authorizeHttpRequests(auth -> 
                auth.anyRequest().hasAuthority("ADMIN")
            )
.formLogin(form -> form
                .loginPage("/admin/login")
                .defaultSuccessUrl("/admin/dashboard")
            )
            .sessionManagement(session -> session
                .sessionFixation().changeSessionId()
                .maximumSessions(1)
            )
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            );

        return http.build();
    }

    /**
     * 5. 일반 웹 페이지 체인 (기본)
     * - 가장 마지막 우선순위
     * - 모든 요청 처리
     */
    @Bean
    @Order(5)
    public SecurityFilterChain defaultWebChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/**")
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/", "/home", "/about", "/register").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .logout(logout -> logout.permitAll());

        return http.build();
    }
}
```

---

### 6-2. 실무 전략: 7.0.2 시대의 보안 설계 원칙

#### ✔ 원칙 1: "필터 체인은 많아져도 된다"

**이유:**
- 각 체인의 책임이 명확해짐
- 설정 충돌 가능성 감소
- 성능 최적화 지점 명확
- 유지보수 용이

**안티패턴:**
```java
// ❌ 하나의 거대한 체인에 모든 로직
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/css/**").permitAll()
    .requestMatchers("/api/public/**").permitAll()
    .requestMatchers("/api/**").authenticated()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    // ... 수십 개의 규칙
);
```

**권장 패턴:**
```java
// ✅ 용도별로 분리된 체인
// - 정적 리소스 체인
// - 공개 API 체인
// - 인증 API 체인
// - 관리자 체인
// - 일반 웹 체인
```

---

#### ✔ 원칙 2: WebSecurity는 최후의 수단

**사용해도 되는 거의 유일한 경우:**

1. **초고빈도 헬스체크**
   ```java
   web.ignoring().requestMatchers("/actuator/health/liveness");
   ```
   - 로드밸런서가 초당 수천 건 호출
   - 보안 헤더 불필요
   - 성능 크리티컬

2. **Servlet Container 레벨 처리**
   - Spring Security가 기술적으로 접근 불가능한 경로

**사용하면 안 되는 경우:**
- 정적 리소스 (→ HttpSecurity.permitAll() 사용)
- 공개 API (→ HttpSecurity.permitAll() 사용)
- "귀찮아서" (→ 절대 안 됨)

---

#### ✔ 원칙 3: 명시하지 않으면 없다 (Explicit Configuration)

Spring Security 7.x는 **암묵적 기본값에 의존하지 않습니다**.

```java
// ❌ 6.x 스타일 (암묵적)
http.authorizeHttpRequests(auth -> 
    auth.anyRequest().authenticated()
);
// CSRF는? 세션은? 헤더는? → "기본값에 맡김"

// ✅ 7.x 스타일 (명시적)
http
    .authorizeHttpRequests(auth -> 
        auth.anyRequest().authenticated()
    )
    .csrf(csrf -> csrf.csrfTokenRepository(...))  // 명시
    .sessionManagement(session -> ...)            // 명시
    .headers(headers -> ...);                     // 명시
```

> ❗ **7.x 철학**  
> "설정하지 않은 보안은 없는 보안"

---

### 6-3. JWT + OAuth2 실전 예제

```java
@Configuration
@EnableWebSecurity
public class OAuth2SecurityConfig {

    /**
     * OAuth2 Resource Server 설정
     * - JWT 검증
     * - Scope 기반 인가
     */
    @Bean
    @Order(1)
    public SecurityFilterChain apiChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**")
                    .hasAuthority("SCOPE_admin")
                .anyRequest()
                    .hasAuthority("SCOPE_read")
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .decoder(jwtDecoder())
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            )
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .csrf(csrf -> csrf.disable());

        return http.build();
    }

    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withJwkSetUri("https://auth.example.com/.well-known/jwks.json")
            .build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter = 
            new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter jwtAuthenticationConverter = 
            new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(
            grantedAuthoritiesConverter
        );
        
        return jwtAuthenticationConverter;
    }
}
```

---

## 7. 결론 및 마이그레이션 가이드

### 7-1. 핵심 요약

Spring Security 7.0.2는 개발자를 불편하게 만들기 위해 바뀐 것이 아닙니다. **"보안은 암묵적이면 반드시 사고가 난다"**는 현실을 코드에 반영한 결과입니다.

**7.0.2의 핵심 메시지:**

1. **HttpSecurity는 유일한 보안 설계 도구**
   - 거의 모든 설정이 여기서 이루어짐
   - Lambda DSL로 명시적 구성 강제

2. **WebSecurity는 거의 사용하지 않아야 할 내부 훅**
   - ignoring()은 보안 책임 포기를 의미
   - 극히 제한적인 경우에만 사용

3. **다중 FilterChain이 정답**
   - 용도별로 체인 분리
   - 명확한 책임 경계

4. **명시적 설정이 안전**
   - 기본값에 의존하지 말 것
   - 모든 보안 정책을 명시

---

### 7-2. 6.x → 7.x 마이그레이션 체크리스트

#### ✅ 필수 변경 사항

- [ ] `.and()` 메서드 제거 및 Lambda DSL로 전환
- [ ] `WebSecurity.ignoring()` → `HttpSecurity.permitAll()` 전환
- [ ] `antMatchers()` → `requestMatchers()` 전환
- [ ] `@EnableGlobalMethodSecurity` → `@EnableMethodSecurity` 전환
- [ ] 모든 보안 설정을 `SecurityFilterChain` 빈으로 전환

#### ✅ 권장 개선 사항

- [ ] 용도별로 `SecurityFilterChain` 분리
- [ ] `securityMatcher()`와 `requestMatchers()` 명확히 구분
- [ ] 모든 보안 정책 명시적 설정
- [ ] 정적 리소스 체인 별도 분리
- [ ] CSRF, 세션, 헤더 정책 명시

#### ✅ 검증 사항

- [ ] 모든 엔드포인트가 의도한 체인을 타는지 확인
- [ ] 보안 헤더가 올바르게 적용되는지 확인
- [ ] CSRF 토큰이 필요한 곳에서 동작하는지 확인
- [ ] 세션 정책이 올바르게 적용되는지 확인

---

### 7-3. 실패 사례와 교훈

#### 실패 사례 1: WebSecurity.ignoring() 남용

```java
// ❌ 마이그레이션 실패
web.ignoring().requestMatchers("/css/**", "/api/public/**");
```

**문제:**
- API 엔드포인트까지 필터 우회
- CORS 정책 미적용으로 크로스 도메인 공격 가능
- 감사 로그 누락으로 컴플라이언스 실패

**해결:**
```java
// ✅ 올바른 마이그레이션
@Bean
@Order(1)
SecurityFilterChain staticChain(HttpSecurity http) {
    http.securityMatcher("/css/**")
        .authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
    return http.build();
}

@Bean
@Order(2)
SecurityFilterChain publicApiChain(HttpSecurity http) {
    http.securityMatcher("/api/public/**")
        .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
        .cors(withDefaults());  // CORS 적용
    return http.build();
}
```

---

#### 실패 사례 2: securityMatcher 미사용

```java
// ❌ 체인 구분 없이 하나로 처리
@Bean
SecurityFilterChain filterChain(HttpSecurity http) {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/**").authenticated()
        .requestMatchers("/admin/**").hasRole("ADMIN")
        // ... 수십 개 규칙
    );
}
```

**문제:**
- API와 웹 페이지의 세션 정책 충돌
- CSRF 설정 불일치
- 유지보수 어려움

**해결:**
```java
// ✅ 체인 분리
@Bean @Order(1)
SecurityFilterChain apiChain(HttpSecurity http) {
    http.securityMatcher("/api/**")
        .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
        .csrf(csrf -> csrf.disable());
    return http.build();
}

@Bean @Order(2)
SecurityFilterChain webChain(HttpSecurity http) {
    http.securityMatcher("/**")
        .sessionManagement(s -> s.sessionCreationPolicy(IF_REQUIRED))
        .csrf(withDefaults());
    return http.build();
}
```

---

### 마무리

`HttpSecurity`는 이제 **유일한 보안 설계 도구**이고, `WebSecurity`는 **거의 사용하지 않아야 할 내부 혹**이 되었습니다.

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