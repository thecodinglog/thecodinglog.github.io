---
layout: post
title: Spring Boot 4와 Servlet 6.1 - 현대적 웹 애플리케이션으로의 진화
description: Spring Boot 4와 Servlet 6.1 - Servlet역사와 Spring과의 관계
date:   2026-01-06 13:00:00 +0900
author: Jeongjin Kim
categories: Spring Spring-boot
tags:	Spring Boot Servlet Servlet6.1
---

# 들어가며

Spring Boot 4가 출시되면서 가장 큰 변화 중 하나는 **Servlet 6.1을 필수로 요구**한다는 점입니다. 이번 글에서는 Servlet의 역사부터 Spring Boot와의 관계, 그리고 최신 변화까지 살펴보겠습니다.

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

# Servlet이란 무엇인가?

Servlet은 Java로 웹 애플리케이션을 만들 때 HTTP 요청을 처리하는 **서버 측 프로그램**입니다. Spring Boot로 API를 만들면, 내부적으로는 Servlet 기술이 동작하고 있습니다. Spring의 `DispatcherServlet`도 결국 Servlet을 확장한 것이죠.

쉽게 말해, Servlet은 웹 서버와 애플리케이션 사이의 **통역사** 역할을 합니다. 사용자의 요청을 받아서 Java 코드로 전달하고, 그 결과를 다시 HTTP 응답으로 변환해주는 거죠.

# Spring Boot와 Servlet의 관계

Spring Boot는 내부적으로 Tomcat, Jetty 같은 **내장 웹 서버(Embedded Container)**를 사용합니다. 이 웹 서버들이 바로 Servlet 컨테이너인데요, Spring Boot 버전이 올라갈 때마다 요구하는 Servlet 버전도 함께 올라갑니다.

### 버전별 매핑 한눈에 보기

| Spring Boot | Spring Framework | 최소 Servlet | Jakarta EE | 지원 컨테이너 |
|-------------|------------------|--------------|------------|---------------|
| 1.x ~ 2.0.x | 4.x ~ 5.x | 3.0 | Java EE 7/8 | Tomcat 7/8, Jetty 8/9 |
| 2.1.x ~ 2.7.x | 5.1.x ~ 5.3.x | 3.1 | Java EE 8 | Tomcat 9/10 |
| 3.0.x ~ 3.3.x | 6.0.x ~ 6.1.x | 6.0 | Jakarta EE 10 | Tomcat 10.1 (Java 17+) |
| **4.0+** | **7.0+** | **6.1** | **Jakarta EE 11** | **Tomcat 11, Jetty 12** |

보시다시피 Spring Boot 4는 Spring Framework 7과 함께 Servlet 6.1을 기반으로 합니다. 이는 단순한 버전업이 아니라, **Jakarta EE 11**이라는 새로운 플랫폼으로의 전환을 의미합니다.

# Servlet의 진화 과정

Servlet은 20년 넘게 꾸준히 발전해왔습니다. 주요 버전별 특징을 살펴보겠습니다.

### Servlet 3.0 (2009년) - 현대화의 시작
- **비동기 처리** 도입: 긴 작업을 처리할 때 쓰레드를 블로킹하지 않고 효율적으로 처리
- **어노테이션 지원**: `@WebServlet`, `@WebFilter` 같은 어노테이션으로 web.xml 없이도 설정 가능
- **파일 업로드 개선**: 멀티파트 업로드를 표준으로 지원

### Servlet 3.1 (2013년) - 성능 강화
- **논블로킹 I/O**: 네트워크 통신을 더욱 효율적으로 처리
- **HTTP 프로토콜 업그레이드**: WebSocket 같은 새로운 프로토콜 지원
- **보안 강화**: 더 안전한 웹 애플리케이션 개발 가능

### Servlet 5.0 (2020년) - 대전환
가장 큰 변화는 **패키지 이름 변경**입니다:
- `javax.servlet` → `jakarta.servlet`

이게 왜 중요할까요? Oracle이 Java EE를 Eclipse 재단에 기부하면서, 상표권 문제로 `javax`를 더 이상 사용할 수 없게 되었습니다. 그래서 모든 패키지 이름을 `jakarta`로 바꾼 거죠. Spring Boot 3부터 이 변화가 적용되었고, 기존 코드에서 import 문을 모두 수정해야 했습니다.

### Servlet 6.0 (2022년) - 현대화
- **Java SE 17 기준**: 최신 Java 기능 활용
- **모듈화 개선**: 더 나은 구조화
- **ByteBuffer 지원**: 메모리 효율적인 I/O 처리
- **쿠키 처리 개선**: 더 유연한 쿠키 설정

### Servlet 6.1 (2024년) - 안정성 강화
Spring Boot 4가 요구하는 최신 버전입니다:

1. **SecurityManager 완전 제거**
   - 오래된 보안 메커니즘인 SecurityManager가 Java에서 deprecated 되면서 Servlet에서도 완전히 제거되었습니다
   - 더 이상 사용하지 않는 코드를 정리해서 경량화

2. **HTTP 상태 코드 및 문자 인코딩 개선**
   - 더 정확한 상태 코드 처리
   - 다국어 처리가 더욱 정교해짐

3. **Charset을 String 대신 직접 사용**
   - 문자 인코딩을 더 타입 안전하게 처리
   - 실수로 잘못된 인코딩 이름을 넣는 것을 방지

4. **ByteBuffer 지원 강화**
   - `ServletInputStream`/`OutputStream`에서 ByteBuffer를 직접 사용 가능
   - 대용량 데이터 처리 시 성능 향상
   - 비동기 I/O가 더욱 효율적으로 동작

# Spring Boot 4에서의 실질적 변화

### 1. 지원 컨테이너 변경
- **사용 가능**: Tomcat 11, Jetty 12
- **제거됨**: Undertow (Servlet 6.1 미지원)

기존에 Undertow를 사용하던 프로젝트는 Tomcat이나 Jetty로 마이그레이션해야 합니다.

### 2. Java 17 이상 필수
Servlet 6.1이 Java 17을 기준으로 하기 때문에, Spring Boot 4도 최소 Java 17을 요구합니다. Java 8이나 11을 쓰던 프로젝트는 업그레이드가 필요합니다.

### 3. Hibernate 7.0 전환
Jakarta EE 11은 JPA 3.2를 포함합니다. 이에 따라 Hibernate도 7.x 버전으로 업그레이드되며, 몇 가지 주의할 점이 있습니다:

- **Dialect 설정 변경**: 데이터베이스별 Dialect 클래스 경로나 설정 방식이 일부 변경될 수 있습니다
- **어노테이션 동작 변경**: JPA 3.2의 새로운 기능과 개선사항이 반영되어 기존 어노테이션의 세부 동작이 달라질 수 있습니다
- **쿼리 최적화**: Hibernate 7은 성능 개선과 함께 쿼리 생성 로직이 일부 변경되었습니다

실무에서는 마이그레이션 전 충분한 테스트, 특히 복잡한 쿼리나 커스텀 Dialect를 사용하는 경우 동작 검증이 필요합니다.

### 4. 가상 스레드(Virtual Threads) 기본 활성화
Spring Boot 4는 Java 21 이상 환경에서 `spring.threads.virtual.enabled=true`를 기본값으로 채택하는 방향입니다. 가상 스레드는 경량 스레드로, 수만 개의 동시 연결을 효율적으로 처리할 수 있습니다.

**주의사항:**
- **ThreadLocal 사용 점검**: 스레드 로컬을 많이 사용하는 라이브러리(레거시 보안 필터, 일부 로깅 프레임워크 등)에서 메모리 누수가 발생할 수 있습니다
- **스레드 풀 설정 재검토**: 기존의 스레드 풀 크기 설정이 가상 스레드 환경에서는 의미가 달라질 수 있습니다
- **호환성 확인**: 사용 중인 서드파티 라이브러리가 가상 스레드를 지원하는지 확인이 필요합니다

### 5. Observability 강화
Servlet 6.1 레이어에서 발생하는 이벤트를 Micrometer가 더 세밀하게 추적합니다:

- **메트릭 명칭 변경**: 대시보드에 표시되는 일부 메트릭 이름이 변경될 수 있어, Grafana나 Prometheus 대시보드 쿼리를 업데이트해야 할 수 있습니다
- **추적 정보 증가**: HTTP 요청/응답에 대한 더 상세한 추적 정보가 수집됩니다
- **성능 오버헤드 최소화**: 강화된 추적 기능에도 불구하고 성능 영향은 최소화되도록 설계되었습니다

모니터링 환경을 구축한 경우, 업그레이드 후 대시보드와 알람 설정을 재검토하는 것이 좋습니다.

### 6. 성능과 안정성 개선
ByteBuffer 지원이 강화되면서 특히 다음 상황에서 성능이 개선됩니다:
- 파일 업로드/다운로드
- 스트리밍 API
- 대용량 데이터 처리
- 실시간 통신

# 마이그레이션 체크리스트

### 버전 업그레이드 시 확인 사항

1. **Java 버전 확인**
   - Spring Boot 4 사용 시 최소 Java 17 필요 (가상 스레드는 Java 21+)
   - `pom.xml` 또는 `build.gradle`에서 Java 버전 설정 확인

2. **의존성 패키지 확인**
   - `javax.servlet` → `jakarta.servlet` 변경 여부 확인
   - Spring Boot 3 이상이면 이미 jakarta로 전환됨

3. **웹 컨테이너 확인**
   - Undertow 사용 중이면 Tomcat이나 Jetty로 변경
   - 기본 내장 Tomcat을 쓴다면 자동으로 Tomcat 11이 사용됨

4. **데이터베이스 레이어 점검**
   - Hibernate Dialect 설정 확인
   - JPA 쿼리 및 네이티브 쿼리 테스트
   - 트랜잭션 동작 검증

5. **가상 스레드 준비**
   - ThreadLocal 사용 패턴 검토
   - 레거시 동기화 코드 확인
   - 성능 테스트 수행

6. **모니터링 환경 업데이트**
   - 메트릭 대시보드 쿼리 확인
   - 알람 임계값 재조정
   - 로그 포맷 변경 여부 확인

7. **테스트 실행**
   - 비동기 처리 부분 테스트
   - 파일 업로드/다운로드 기능 테스트
   - 문자 인코딩 관련 기능 테스트
   - 부하 테스트로 성능 검증

# 실무 관점에서의 접근

대부분의 Spring Boot 애플리케이션은 Servlet을 직접 다루지 않습니다. `@RestController`, `@RequestMapping` 같은 Spring MVC 어노테이션을 사용하죠. 하지만 내부적으로는 모두 Servlet 위에서 동작합니다.

```java
// 일반적으로 작성하는 코드
@RestController
public class UserController {
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.findAll();
    }
}

// 내부적으로는 DispatcherServlet이 이 요청을 처리
// DispatcherServlet → Servlet 6.1 기반으로 동작
```

Servlet 버전이 올라가면서 이런 내부 동작이 더 빠르고 안정적으로 개선됩니다. 특히 가상 스레드와 결합되면, 동일한 하드웨어에서 훨씬 많은 동시 요청을 처리할 수 있습니다.

# 정리하며

Spring Boot 4의 Servlet 6.1 요구사항은 단순한 버전업이 아닙니다. **Java 생태계 전체의 현대화** 과정이죠:

- Java EE → Jakarta EE로의 전환
- 오래된 코드 제거 (SecurityManager 등)
- 최신 Java 기능 활용 (Java 17+, 가상 스레드)
- 성능 개선 (ByteBuffer, 비동기 I/O)
- 데이터 접근 계층 현대화 (Hibernate 7.0, JPA 3.2)
- 운영 가시성 향상 (Observability 강화)

**핵심 요약:**
- Spring Boot 4 = Spring Framework 7 + Servlet 6.1 + Jakarta EE 11
- Java 17 최소, Java 21 권장 (가상 스레드 활용)
- Hibernate 7.0 전환으로 JPA 레이어 개선
- 더 빠르고, 더 안정적이며, 더 관찰 가능한 애플리케이션 구축 가능

프로덕션 환경에 적용하기 전에는 충분한 테스트와 성능 검증을 거치시길 권장합니다. Spring 공식 문서와 마이그레이션 가이드를 참고하면 더 원활한 전환이 가능할 것입니다. 🚀

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