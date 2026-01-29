---
layout: post
title: Spring Security 7.0 경로 매칭 변경 사항 정리
description: Spring Security 7.0 경로 매칭 변경 사항 정리
date:   2026-01-29 16:40:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	Spring Security Url Matcher
---
# Spring Security 7.0 경로 매칭 변경 사항 정리

Spring Security 7.0으로 업그레이드하면서 기존 코드가 동작하지 않아 당황하셨나요? 바로 경로 매칭 방식이 변경되었기 때문입니다. 이번 포스트에서는 무엇이 바뀌었고, 어떻게 수정해야 하는지 정리해보겠습니다.

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

## 무엇이 바뀌었나?

Spring Security 7.0부터는 URL 매칭 방식에 큰 변화가 있었습니다. 기존에 사용하던 `AntPathRequestMatcher`와 `MvcRequestMatcher`가 제거되고, `PathPatternRequestMatcher`로 대체되었습니다.

### 제거된 Matcher
- `AntPathRequestMatcher`
- `MvcRequestMatcher`

### 새롭게 추가된 Matcher
- `PathPatternRequestMatcher`

## 왜 바뀌었을까?

이러한 변경에는 몇 가지 이유가 있습니다.

첫째, Spring Framework 5.3부터 도입된 PathPatternParser 기반으로 URL 매칭 성능을 최적화하기 위함입니다. 기존 Ant 방식보다 더 빠르고 효율적인 매칭이 가능해졌습니다.

둘째, URL 패턴 문법을 통일하고 더 명확하게 처리하기 위해서입니다. 이제 Spring Security는 기본적으로 PathPatternRequestMatcher 방식으로 경로를 매칭합니다.

## 코드 수정 방법

그렇다면 기존 코드를 어떻게 수정해야 할까요? 두 가지 방법이 있습니다.

### 방법 A: PathPatternRequestMatcher 사용 (권장)

가장 명시적이고 공식적인 방식입니다.

```java
import org.springframework.security.web.servlet.util.matcher.PathPatternRequestMatcher;

@Bean
@Order(20)
public SecurityFilterChain apiAdminSecurityFilterChain(HttpSecurity http) throws Exception {
    http
        // 수정됨: AntPathRequestMatcher → PathPatternRequestMatcher
        .securityMatcher(PathPatternRequestMatcher.pathPattern("/api/**"))
        
        .csrf(AbstractHttpConfigurer::disable)
        .cors(Customizer.withDefaults());
        // ... (나머지 설정 동일)

    return http.build();
}
```

### 방법 B: 문자열 패턴 직접 사용 (더 간결)

`HttpSecurity.securityMatcher(String... patterns)`는 내부적으로 `PathPatternRequestMatcher`를 사용하도록 변경되었습니다. 따라서 객체를 직접 생성하지 않고 문자열만 넘겨도 됩니다.

```java
@Bean
@Order(20)
public SecurityFilterChain apiAdminSecurityFilterChain(HttpSecurity http) throws Exception {
    http
        // 수정됨: 문자열 패턴 전달 (내부적으로 PathPatternRequestMatcher 사용)
        .securityMatcher("/api/**")
        
        .csrf(AbstractHttpConfigurer::disable);
        // ...

    return http.build();
}
```

`/api/**`처럼 단순한 패턴은 방법 B가 가장 깔끔합니다.

## PathPattern 문법 주의사항

PathPattern은 Ant 스타일과 유사하지만 문법이 더 엄격합니다. 특히 `**` (이중 와일드카드) 사용 시 주의해야 합니다.

### `**` 규칙

이중 와일드카드는 **반드시 패턴의 맨 끝에서만 사용 가능**합니다.

| 패턴 | 가능 여부 |
|------|-----------|
| `/api/**` | ✅ 가능 |
| `/api/**/details` | ❌ 불가능 |

만약 `/api/**/details` 같은 중간 와일드카드가 필요하다면 `RegexRequestMatcher` 사용을 고려해야 합니다.

## 정리

Spring Security 7.0으로 업그레이드할 때 기억해야 할 핵심 사항은 다음과 같습니다.

- `AntPathRequestMatcher`는 더 이상 사용할 수 없습니다
- `PathPatternRequestMatcher`가 공식 대체 방식입니다
- 단순 패턴(`/api/**`)은 문자열 방식이 가장 간단하고 권장됩니다
- `**`는 패턴의 맨 끝에서만 사용 가능하므로 주의가 필요합니다

마이그레이션 과정에서 이 내용을 참고하시면 큰 어려움 없이 업그레이드하실 수 있을 것입니다.

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