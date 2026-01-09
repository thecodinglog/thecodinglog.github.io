---
layout: post
title: 스프링 시큐리티, 이 글 하나로 기억 되살리기 🔐
description: 스프링 시큐리티, 이 글 하나로 기억 되살리기 🔐
date:   2026-01-09 13:00:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	Spring Security
---

오랜만에 스프링 시큐리티 설정을 만지려고 하면 "어, 이게 뭐였지?" 싶은 순간이 오죠. 필터 체인이니 뭐니 하는 용어들이 머릿속에서 뒤죽박죽 섞여버립니다. 이 글은 그런 분들을 위해 스프링 시큐리티 7의 핵심 아키텍처를 쉽게 정리했습니다.

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

# 시작하기 전에: 서블릿 필터부터 이해하자

스프링 시큐리티를 이해하려면 먼저 서블릿 필터가 뭔지 알아야 합니다. 

클라이언트가 HTTP 요청을 보내면, 서블릿 컨테이너는 `FilterChain`을 만듭니다. 이 체인에는 여러 `Filter` 인스턴스들과 최종 목적지인 `Servlet`(스프링 MVC에서는 `DispatcherServlet`)이 포함되어 있죠.

필터는 두 가지 일을 할 수 있습니다:
- 요청을 가로채서 뒤에 있는 필터나 서블릿이 실행되지 않게 막기 (주로 응답을 직접 작성)
- 요청이나 응답을 수정해서 다음 단계로 넘기기

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response,
                     FilterChain chain) throws IOException, ServletException {
    // 애플리케이션 실행 전에 뭔가 하고
    chain.doFilter(request, response); // 다음 단계로 넘기고
    // 애플리케이션 실행 후에 뭔가 하기
}
```

여기서 중요한 점: **필터의 순서가 매우 중요합니다.** 필터는 앞에서부터 순차적으로 실행되고, 각 필터는 자기 뒤에 있는 것들에만 영향을 주니까요.

# DelegatingFilterProxy: 스프링과 서블릿 컨테이너를 연결하는 다리

여기서 문제가 하나 있습니다. 서블릿 컨테이너는 자체 표준으로 필터를 등록하는데, 스프링 빈에 대해서는 모릅니다. 

`DelegatingFilterProxy`가 바로 이 문제를 해결합니다. 서블릿 컨테이너에는 `DelegatingFilterProxy`를 등록하고, 실제 작업은 스프링 빈으로 등록된 필터에게 위임하는 거죠.

```java
public void doFilter(ServletRequest request, ServletResponse response,
                     FilterChain chain) {
    Filter delegate = getFilterBean(someBeanName); // 스프링 빈 가져오기
    delegate.doFilter(request, response); // 실제 작업 위임
}
```

이렇게 하면 필터 빈을 지연 로딩할 수 있다는 장점도 있습니다. 서블릿 컨테이너는 필터를 먼저 등록해야 하는데, 스프링 빈은 그보다 나중에 로드되거든요.

# FilterChainProxy: 스프링 시큐리티의 핵심

스프링 시큐리티의 서블릿 지원은 모두 `FilterChainProxy` 안에 들어있습니다. 이 녀석은 스프링 시큐리티가 제공하는 특별한 필터로, 여러 필터 인스턴스에게 작업을 위임할 수 있습니다. `FilterChainProxy`는 빈이기 때문에 보통 `DelegatingFilterProxy`로 감싸져 있죠.

`FilterChainProxy`가 제공하는 장점들:
- 스프링 시큐리티의 서블릿 지원 시작점 역할 (디버깅할 때 여기에 브레이크포인트 걸면 좋음)
- `SecurityContext`를 정리해서 메모리 누수 방지
- `HttpFirewall`을 적용해서 특정 공격으로부터 보호
- URL뿐만 아니라 `HttpServletRequest`의 다른 요소들도 고려해서 필터 체인 결정

# SecurityFilterChain: 실제 필터들의 집합

`SecurityFilterChain`은 `FilterChainProxy`가 현재 요청에 대해 어떤 스프링 시큐리티 필터들을 실행할지 결정하는 데 사용됩니다.

하나의 애플리케이션에 여러 개의 `SecurityFilterChain`이 있을 수 있습니다. `FilterChainProxy`는 어떤 체인을 사용할지 결정하는데, **처음으로 매칭되는 체인만 실행됩니다.**

예를 들어:
- `/api/messages/` 요청이 오면 → `/api/**` 패턴의 `SecurityFilterChain0`에 먼저 매칭되어 실행
- `/messages/` 요청이 오면 → `SecurityFilterChain0`에 안 맞으니 다음 체인들을 시도

각 `SecurityFilterChain`은 독립적으로 설정할 수 있고, 포함된 필터의 개수도 다를 수 있습니다. 심지어 필터가 0개인 체인도 만들 수 있어요 (특정 요청을 스프링 시큐리티가 무시하게 하려면).

# Security Filters: 실행 순서가 중요하다

보안 필터들은 특정 순서대로 실행됩니다. 예를 들어 인증을 수행하는 필터가 인가를 수행하는 필터보다 먼저 실행되어야겠죠?

다음 설정을 보세요:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults())
            .authorizeHttpRequests((authorize) -> authorize
                .anyRequest().authenticated()
            );
        return http.build();
    }
}
```

이 설정은 다음 순서로 필터를 실행합니다:

1. `CsrfFilter` - CSRF 공격 방어
2. `BasicAuthenticationFilter` - HTTP Basic 인증
3. `UsernamePasswordAuthenticationFilter` - 폼 로그인 인증
4. `AuthorizationFilter` - 인가 처리

# 실전 설정: 기본부터 시작하기

가장 기본적인 스프링 시큐리티 설정은 이렇습니다:

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withDefaultPasswordEncoder()
            .username("user")
            .password("password")
            .roles("USER")
            .build());
        return manager;
    }
}
```

이 간단한 설정만으로도:
- 모든 URL에 대해 인증 요구
- 로그인 폼 자동 생성
- 폼 기반 인증 지원
- 로그아웃 지원
- CSRF 공격 방어
- 세션 고정 공격 방어
- 보안 헤더 통합 (HSTS, X-Content-Type-Options 등)
- 서블릿 API 메서드 통합

이 모든 게 작동합니다!

# HttpSecurity: 세밀한 제어

위의 기본 설정은 사실 내부적으로 이런 `SecurityFilterChain`을 만듭니다:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests((authorize) -> authorize
            .anyRequest().authenticated()
        )
        .formLogin(Customizer.withDefaults())
        .httpBasic(Customizer.withDefaults());
    return http.build();
}
```

이 설정은:
- 모든 요청에 인증 필요
- 폼 로그인 허용
- HTTP Basic 인증 허용

# 여러 개의 HttpSecurity: 영역별로 다른 보안 설정

실제 애플리케이션에서는 영역마다 다른 보안 설정이 필요할 때가 많습니다. 여러 개의 `SecurityFilterChain` 빈을 등록하면 됩니다:

```java
@Configuration
@EnableWebSecurity
public class MultiHttpSecurityConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        UserBuilder users = User.withDefaultPasswordEncoder();
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(users.username("user").password("password").roles("USER").build());
        manager.createUser(users.username("admin").password("password").roles("USER","ADMIN").build());
        return manager;
    }

    @Bean
    @Order(1)  // 우선순위 높음
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")  // /api/**에만 적용
            .authorizeHttpRequests((authorize) -> authorize
                .anyRequest().hasRole("ADMIN")
            )
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }

    @Bean  // @Order 없으면 가장 낮은 우선순위
    public SecurityFilterChain formLoginFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests((authorize) -> authorize
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());
        return http.build();
    }
}
```

# securityMatcher vs requestMatchers: 뭐가 다른가?

헷갈리기 쉬운 부분입니다:

- `http.securityMatcher()`: 이 `SecurityFilterChain` 자체가 어떤 요청에 적용될지 결정
- `requestMatchers()`: 체인 안에서 개별 인가 규칙이 어떤 요청에 적용될지 결정

```java
@Bean
public SecurityFilterChain securedFilterChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/secured/**")  // 이 체인은 /secured/**에만 적용
        .authorizeHttpRequests((authorize) -> authorize
            .requestMatchers("/secured/user").hasRole("USER")    // 세부 규칙
            .requestMatchers("/secured/admin").hasRole("ADMIN")  // 세부 규칙
            .anyRequest().authenticated()
        )
        .httpBasic(Customizer.withDefaults())
        .formLogin(Customizer.withDefaults());
    return http.build();
}
```

**중요:** `securityMatcher`를 지정하면 매칭되는 요청만 보호됩니다. 매칭 안 되는 요청은 스프링 시큐리티가 보호하지 않아요! 따라서 `securityMatcher` 없는 기본 체인을 하나 두는 게 좋습니다.

# SecurityFilterChain 엔드포인트 주의사항

필터 체인이 제공하는 엔드포인트(예: `/login`, `/logout`)는 `securityMatcher`의 영향을 자동으로 받지 않습니다:

```java
@Bean
@Order(1)
public SecurityFilterChain securedFilterChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/secured/**")
        .authorizeHttpRequests((authorize) -> authorize
            .anyRequest().authenticated()
        )
        .formLogin((formLogin) -> formLogin
            .loginPage("/secured/login")           // 커스텀 경로
            .loginProcessingUrl("/secured/login")  // 커스텀 경로
            .permitAll()
        )
        .logout((logout) -> logout
            .logoutUrl("/secured/logout")
            .logoutSuccessUrl("/secured/login?logout")
            .permitAll()
        );
    return http.build();
}

@Bean
public SecurityFilterChain defaultFilterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests((authorize) -> authorize
            .anyRequest().denyAll()  // 나머지는 모두 거부
        );
    return http.build();
}
```

# 실전 예제: 뱅킹 시스템

마지막으로 실제 서비스처럼 복잡한 예제를 봅시다:

```java
@Configuration
@EnableWebSecurity
public class BankingSecurityConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        UserBuilder users = User.withDefaultPasswordEncoder();
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(users.username("user1").password("password").roles("USER", "VIEW_BALANCE").build());
        manager.createUser(users.username("user2").password("password").roles("USER").build());
        manager.createUser(users.username("admin").password("password").roles("ADMIN").build());
        return manager;
    }

    @Bean
    @Order(1)  // 가장 높은 우선순위
    public SecurityFilterChain approvalsSecurityFilterChain(HttpSecurity http) throws Exception {
        String[] approvalsPaths = { 
            "/accounts/approvals/**", 
            "/loans/approvals/**", 
            "/credit-cards/approvals/**" 
        };
        http
            .securityMatcher(approvalsPaths)
            .authorizeHttpRequests((authorize) -> authorize
                .anyRequest().hasRole("ADMIN")
            )
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }

    @Bean
    @Order(2)  // 두 번째 우선순위
    public SecurityFilterChain bankingSecurityFilterChain(HttpSecurity http) throws Exception {
        String[] bankingPaths = { "/accounts/**", "/loans/**", "/credit-cards/**", "/balances/**" };
        String[] viewBalancePaths = { "/balances/**" };
        http
            .securityMatcher(bankingPaths)
            .authorizeHttpRequests((authorize) -> authorize
                .requestMatchers(viewBalancePaths).hasRole("VIEW_BALANCE")
                .anyRequest().hasRole("USER")
            );
        return http.build();
    }

    @Bean  // 기본 체인 (가장 낮은 우선순위)
    public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        String[] allowedPaths = { "/", "/user-login", "/user-logout", "/notices", "/contact", "/register" };
        http
            .authorizeHttpRequests((authorize) -> authorize
                .requestMatchers(allowedPaths).permitAll()
                .anyRequest().authenticated()
            )
            .formLogin((formLogin) -> formLogin
                .loginPage("/user-login")
                .loginProcessingUrl("/user-login")
            )
            .logout((logout) -> logout
                .logoutUrl("/user-logout")
                .logoutSuccessUrl("/?logout")
            );
        return http.build();
    }
}
```

이 설정의 로직:

1. **승인 관련 경로** (`@Order(1)`): `/accounts/approvals/**`, `/loans/approvals/**`, `/credit-cards/approvals/**`는 ADMIN 권한 필요, HTTP Basic 인증 사용

2. **뱅킹 경로** (`@Order(2)`): `/accounts/**`, `/loans/**`, `/credit-cards/**`, `/balances/**` 중에서
   - `/balances/**`는 `VIEW_BALANCE` 권한 필요
   - 나머지는 `USER` 권한 필요
   - `/approvals/`가 포함된 요청은 이미 첫 번째 체인에서 처리되므로 여기선 안 걸림

3. **기본 경로** (우선순위 가장 낮음):
   - `/`, `/user-login`, `/user-logout`, `/notices`, `/contact`, `/register`는 인증 없이 접근 가능
   - 나머지는 인증 필요
   - 폼 로그인 사용

# 마무리하며

스프링 시큐리티의 핵심을 정리하면:

1. **필터 체인 기반 아키텍처**: 서블릿 필터를 기반으로 작동
2. **DelegatingFilterProxy**: 서블릿 컨테이너와 스프링을 연결
3. **FilterChainProxy**: 여러 SecurityFilterChain 중 적절한 것을 선택
4. **SecurityFilterChain**: 실제 보안 필터들의 집합
5. **우선순위**: `@Order`로 체인 실행 순서 제어
6. **매처 구분**: `securityMatcher`(체인 적용 범위) vs `requestMatchers`(개별 규칙)

이 글이 오랜만에 스프링 시큐리티를 만지는 분들께 도움이 되길 바랍니다. 기억이 가물가물할 때 다시 읽어보세요! 🚀

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