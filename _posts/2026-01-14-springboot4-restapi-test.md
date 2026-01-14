---
layout: post
title: Spring Boot 4 REST API 테스팅, 이 글 하나로 정리하기 🧪
description: Spring Boot 4 REST API 테스팅, 이 글 하나로 정리하기 🧪
date:   2026-01-14 14:00:00 +0900
author: Jeongjin Kim
categories: Spring Spring-boot
tags:	SpringBoot Spring-boot RESTAPI테스트
---

Spring Boot 4가 출시되면서 REST API 테스팅 방식이 또 바뀌었습니다. MockMvc는 장황하고, TestRestTemplate은 곧 사라질 예정이고, WebTestClient는 리액티브 의존성이 필요하죠. "도대체 뭘 써야 하나?" 싶으셨다면, 이 글이 답입니다. Spring Framework 7의 새로운 표준, **RestTestClient**를 완벽히 정리했습니다.

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

# 왜 또 새로운 테스팅 도구인가?

2025년 11월, Spring Boot 4.0과 Spring Framework 7.0이 정식 출시되었습니다. 지난 10년간 자바 웹 개발은 안정성과 확장성을 중심으로 발전했지만, 테스팅 도구는 파편화되어 있었습니다.

## 기존 도구들의 문제점

### MockMvc: 빠르지만 장황하다

MockMvc는 서블릿 컨테이너를 띄우지 않고 DispatcherServlet을 직접 호출해서 빠릅니다. 하지만 API가 정적 빌더 패턴에 의존적이고 장황하죠.

```java
mockMvc.perform(get("/api/books/1"))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.title").value("Spring Boot 4"));
```

게다가 실제 HTTP 통신을 하지 않아서 네트워크 레이어의 직렬화/역직렬화 문제나 프록시 이슈를 감지하지 못합니다.

### TestRestTemplate: 곧 사라질 레거시

실제 서버를 띄워서 통합 테스트를 하는 TestRestTemplate은 RestTemplate 기반의 명령형 스타일입니다.

```java
ResponseEntity<Book> response = restTemplate.getForEntity("/api/books/1", Book.class);
assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
assertThat(response.getBody().getTitle()).isEqualTo("Spring Boot 4");
```

ResponseEntity를 수동으로 언래핑하고 별도의 어설션 라이브러리로 검증해야 하는 번거로움이 있습니다. 게다가 Spring 6.1 이후 RestTemplate이 유지보수 모드에 진입했고, 향후 Deprecated될 예정입니다.

### WebTestClient: 좋지만 의존성 문제

리액티브 스택(WebFlux)용으로 설계된 WebTestClient는 직관적인 Fluent API를 제공합니다.

```java
webTestClient.get().uri("/api/books/1")
    .exchange()
    .expectStatus().isOk()
    .expectBody(Book.class)
    .value(book -> assertThat(book.getTitle()).isEqualTo("Spring Boot 4"));
```

개발자들이 좋아했지만 문제가 있습니다. Spring MVC 애플리케이션을 테스트하는데도 불필요한 리액티브 의존성(Reactor Netty 등)을 추가해야 하거나 클래스패스 충돌이 발생할 수 있습니다.

---

# RestTestClient: 모든 문제를 해결하는 통합 인터페이스

RestTestClient는 이 모든 파편화를 해결하기 위해 설계되었습니다. 내부적으로 동기식 HTTP 클라이언트인 RestClient를 래핑하면서도, 테스트 검증을 위한 풍부한 어설션 API를 제공합니다.

**가장 큰 특징: 단일 API로 모든 테스팅 시나리오를 커버합니다.**

`bindTo` 메서드로 실행 전략(Mock vs Real Server)만 바꾸면, 요청을 구성하고 응답을 검증하는 API는 동일하게 유지됩니다.

```java
// Mock 환경
RestTestClient client = RestTestClient.bindToController(new BookController()).build();

// 실제 서버 환경 (같은 API!)
RestTestClient client = RestTestClient.bindToServer("http://localhost:8080").build();

// 사용법은 동일
client.get().uri("/api/books/1")
    .exchange()
    .expectStatus().isOk()
    .expectBody(Book.class)
    .value(book -> assertThat(book.getTitle()).isEqualTo("Spring Boot 4"));
```

단위 테스트부터 E2E 테스트까지 일관된 문법으로 작성할 수 있어 학습 곡선이 낮고 테스트 코드의 가독성이 비약적으로 향상됩니다.

---

# RestTestClient 아키텍처: 바인딩 전략이 핵심

RestTestClient의 설계 철학은 '유창함(Fluency)'과 '조합성(Composability)'입니다. 단순히 메서드 체이닝을 넘어, 테스트의 의도를 명확히 드러내는 도메인 특화 언어(DSL)에 가깝습니다.

## 바인딩 메서드: 실행 엔진을 선택하라

RestTestClient 인스턴스는 `RestTestClient.bindToXxx()` 형태의 정적 팩토리 메서드로 생성합니다. 이 바인딩 단계에서 요청을 처리할 '백엔드 엔진'이 결정됩니다.

| 바인딩 메서드 | 내부 처리 엔진 | 주요 용도 | 비고 |
|------------|-------------|---------|------|
| `bindToController` | Standalone MockMvc | 단위 테스트, 라우팅 검증 | 컨테이너 로딩 없음 (초고속) |
| `bindToMockMvc` | MockMvc Wrapper | 레거시 마이그레이션, 보안 검증 | 기존 MockMvc 설정 재사용 |
| `bindToApplicationContext` | Full Context MockMvc | 통합 테스트, 빈 상호작용 검증 | 컨텍스트 로딩 필요 |
| `bindToServer` | RestClient (Network) | E2E 테스트, 실제 HTTP 통신 | 실제 서버 구동 필요 |
| `bindToRouterFunction` | RouterFunction Invoker | 함수형 엔드포인트 테스트 | WebFlux/MVC 함수형 스타일 |

이 설계는 "Write Once, Run Anywhere"처럼, 동일한 테스트 로직을 다양한 실행 환경에 적용할 수 있는 유연성을 제공합니다.

## 요청-교환-검증 흐름 (Request-Exchange-Assert)

RestTestClient 사용 흐름은 세 단계로 구분됩니다.

### 1. Request Specification (요청 명세)

HTTP 메서드, URI, 헤더, 바디를 설정합니다. RestClient API와 거의 유사합니다.

```java
client.post().uri("/users")
    .contentType(MediaType.APPLICATION_JSON)
    .body(newUser)
```

### 2. Exchange (교환)

`exchange()` 메서드를 호출하여 요청을 실행하고 응답 컨텍스트로 전환합니다. 이 메서드 명명은 WebTestClient와의 통일성을 위해 선택되었습니다.

### 3. Response Assertion (응답 검증)

반환된 ResponseSpec을 통해 상태 코드, 헤더, 바디를 체이닝 방식으로 검증합니다.

```java
.exchange()
    .expectStatus().isCreated()
    .expectHeader().contentType(MediaType.APPLICATION_JSON)
    .expectBody(User.class).isEqualTo(expectedUser);
```

---

# 바인딩 전략 심층 분석: 언제 무엇을 쓸까?

각 바인딩 전략의 내부 동작 원리와 최적 사용 시나리오를 이해해야 RestTestClient를 효과적으로 활용할 수 있습니다.

## bindToController: 초고속 단위 테스트

스프링 컨테이너 전체를 로딩하지 않고, 필요한 컨트롤러 인스턴스만 등록하여 테스트합니다.

```java
@Test
void testGetBook() {
    BookService mockService = mock(BookService.class);
    when(mockService.getBook(1L)).thenReturn(new Book(1L, "Spring Boot 4"));
    
    RestTestClient client = RestTestClient
        .bindToController(new BookController(mockService))
        .build();
    
    client.get().uri("/api/books/1")
        .exchange()
        .expectStatus().isOk()
        .expectBody(Book.class)
        .value(book -> assertThat(book.title()).isEqualTo("Spring Boot 4"));
}
```

**동작 원리:** 내부적으로 `MockMvcBuilders.standaloneSetup()`을 사용합니다. 스프링의 전체 빈 생명주기를 거치지 않고, 수동으로 생성된 컨트롤러 객체와 최소한의 MVC 인프라(Converter, Resolver)만 구성하여 가상의 DispatcherServlet을 구동합니다.

**장점:** 실행 속도가 매우 빠릅니다. 서비스 계층이나 데이터 액세스 계층을 Mockito로 모킹하여 컨트롤러 로직(요청 매핑, 파라미터 바인딩, 리턴 값 처리)에만 집중할 수 있습니다.

**한계:** `@ControllerAdvice`나 전역 필터, 보안 설정 등이 자동으로 적용되지 않으므로, 이를 테스트하려면 수동으로 추가 설정해야 합니다.

## bindToMockMvc: 레거시와 모던의 가교

이미 MockMvc를 사용하여 복잡한 설정을 구축해 둔 프로젝트라면, `bindToMockMvc`가 가장 현실적인 마이그레이션 경로입니다.

```java
@SpringBootTest
@AutoConfigureMockMvc
class BookControllerTest {
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void testWithSecurity() {
        RestTestClient client = RestTestClient.bindTo(mockMvc).build();
        
        client.get().uri("/api/books/1")
            .exchange()
            .expectStatus().isOk();
    }
}
```

**동작 원리:** 기존에 구성된 MockMvc 인스턴스를 RestTestClient가 감싸는(Wrapper) 형태입니다. 요청 실행은 MockMvc가 담당하고, API 인터페이스만 RestTestClient의 유창한 스타일을 사용합니다.

**활용 전략:** Spring Security 설정, 커스텀 필터 체인, 복잡한 인코딩 설정이 포함된 기존 MockMvc 빌더를 그대로 재사용하면서, 테스트 코드의 가독성만 개선할 수 있습니다. 특히 보안 검증 시 `SecurityMockMvcConfigurers`와의 호환성이 보장되므로 권장됩니다.

## bindToApplicationContext: 완벽한 통합 테스트

실제 애플리케이션과 가장 유사한 환경을 Mock 기반으로 제공합니다.

```java
@SpringBootTest
class BookIntegrationTest {
    @Autowired
    private WebApplicationContext context;
    
    @Test
    void testFullIntegration() {
        RestTestClient client = RestTestClient
            .bindToApplicationContext(context)
            .build();
        
        client.post().uri("/api/books")
            .contentType(MediaType.APPLICATION_JSON)
            .body(new Book(null, "New Book"))
            .exchange()
            .expectStatus().isCreated();
    }
}
```

**동작 원리:** `@SpringJUnitConfig` 또는 `@SpringBootTest`를 통해 로딩된 WebApplicationContext를 주입받아 전체 빈을 스캔하고 구성합니다.

**Context Pausing (Spring 7 신기능):** Spring Framework 7에서는 컨텍스트 캐싱 기능이 개선되어, 사용되지 않는 컨텍스트를 일시 중지(Pause)시키는 기능이 도입되었습니다. 이는 대규모 통합 테스트 슈트 실행 시 메모리 사용량을 획기적으로 줄여주며, `bindToApplicationContext` 사용 시의 리소스 부담을 완화해줍니다.

**장점:** 실제 빈들이 모두 연결되어 동작하므로, 컨트롤러에서 서비스, 리포지토리로 이어지는 데이터 흐름을 검증하기에 적합합니다.

## bindToServer: 진짜 E2E 테스트

RestTestClient의 진정한 강력함을 보여주는 모드입니다. Mock이 아닌 실제 HTTP 통신을 수행합니다.

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class BookE2ETest {
    @LocalServerPort
    private int port;
    
    @Test
    void testRealServer() {
        RestTestClient client = RestTestClient
            .bindToServer("http://localhost:" + port)
            .build();
        
        client.get().uri("/api/books/1")
            .exchange()
            .expectStatus().isOk()
            .expectBody(Book.class)
            .value(book -> assertThat(book.title()).isEqualTo("Spring Boot 4"));
    }
}
```

**동작 원리:** 내부적으로 JDK HttpClient, Apache HttpClient, 또는 Jetty Client 등을 사용하는 RestClient를 통해 실제 네트워크 소켓으로 요청을 전송합니다.

**비교 우위:** MockMvc는 서블릿 컨테이너(Tomcat/Jetty)의 동작(필터, 에러 페이지 렌더링, 실제 HTTP 헤더 처리)을 완벽하게 모사하지 못합니다. 반면 `bindToServer`는 실제 서버와 통신하므로, 네트워크 타임아웃, 프록시 설정, SSL/TLS 핸드셰이크, 실제 JSON 직렬화/역직렬화 호환성 등을 완벽하게 검증할 수 있습니다.

**Virtual Threads:** Spring Boot 4의 Java 21+ 지원과 맞물려, `bindToServer`를 사용하는 테스트 클라이언트는 가상 스레드(Virtual Threads)를 활용하여 블로킹 I/O 성능을 극대화할 수 있습니다.

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

# 마이그레이션 가이드: TestRestTemplate → RestTestClient

Spring Boot 4.0으로의 업그레이드는 TestRestTemplate을 제거하고 RestTestClient로 전환하는 작업을 수반합니다. TestRestTemplate은 향후 버전에서 제거될 예정이므로 조기 마이그레이션이 권장됩니다.

## 의존성 관리 및 모듈화

Spring Boot 4.0은 전체 코드베이스의 모듈화를 단행했습니다. RestTestClient를 사용하려면 올바른 의존성 설정이 필수입니다.

**모듈 분리:** RestTestClient는 `org.springframework.boot:spring-boot-resttestclient` 모듈에 포함되어 있습니다. 런타임 시 동기식 HTTP 클라이언트 기능을 위해 `org.springframework.boot:spring-boot-restclient` 의존성이 필요합니다.

**스타터 패키지:** 일반적으로 `spring-boot-starter-test`에 포함되어 제공되지만, 특정 의존성을 제외하거나 커스텀 스타터를 구성하는 경우 명시적으로 추가해야 할 수 있습니다.

**레거시 지원:** 마이그레이션 과도기 동안 기존 테스트가 깨지는 것을 방지하기 위해, `@AutoConfigureTestRestTemplate` 어노테이션으로 기존 TestRestTemplate 빈을 활성화할 수 있습니다. 새로운 클라이언트는 `@AutoConfigureRestTestClient`를 통해 주입받습니다.

## 코드 변환 패턴

기존의 명령형 코드를 선언형 코드로 변환하는 패턴을 익히면 마이그레이션 효율이 높아집니다.

### 예제 1: 단순 GET 요청

**Legacy (TestRestTemplate):**
```java
ResponseEntity<String> response = restTemplate.getForEntity("/api/books/1", String.class);
assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
```

**Modern (RestTestClient):**
```java
client.get().uri("/api/books/1")
    .exchange()
    .expectStatus().isOk();
```

코드가 간결해지고, '요청'과 '검증'이 하나의 흐름으로 연결되어 가독성이 향상됩니다.

### 예제 2: 복잡한 JSON 응답 검증

**Legacy:**
```java
Book book = restTemplate.getForObject("/api/books/1", Book.class);
assertThat(book.getTitle()).isEqualTo("Spring Boot 4");
assertThat(book.getAuthor()).isEqualTo("Expert");
```

**Modern:**
```java
client.get().uri("/api/books/1")
    .exchange()
    .expectStatus().isOk()
    .expectBody(Book.class)
    .value(book -> {
        assertThat(book.title()).isEqualTo("Spring Boot 4");
        assertThat(book.author()).isEqualTo("Expert");
    });
```

`value()` 메서드를 통해 람다 표현식 내부에서 AssertJ를 활용하므로, 객체 추출 과정을 명시적으로 작성할 필요 없이 직관적인 검증이 가능합니다.

## 마이그레이션 주의사항

**의존성 충돌:** Spring Boot 4.0 업그레이드 시, 이전에 Deprecated되었던 메서드나 클래스가 완전히 삭제되었으므로, 3.x 버전의 최신 릴리스(3.5.x)에서 경고를 먼저 해결하고 넘어가는 것이 안전합니다.

**컴파일 오류:** TestRestTemplate 관련 컴파일 오류 발생 시, 테스트 스코프 의존성(`spring-boot-resttestclient`)이 올바르게 추가되었는지 확인해야 합니다.

**제네릭 타입 처리:** RestTestClient는 `ParameterizedTypeReference`를 지원하여 `List<T>`와 같은 제네릭 타입의 응답 본문을 더욱 안전하게 처리할 수 있습니다.

```java
client.get().uri("/api/books")
    .exchange()
    .expectStatus().isOk()
    .expectBody(new ParameterizedTypeReference<List<Book>>() {})
    .value(books -> assertThat(books).hasSize(3));
```

---

# 보안 테스팅: CSRF와 인증 통합

Spring Security 7과 결합된 환경에서 RestTestClient를 사용하는 방법은 바인딩 전략에 따라 크게 달라집니다. 보안 테스트는 애플리케이션 무결성을 보장하는 가장 중요한 단계입니다.

## MockMvc 바인딩: Mock 환경에서의 보안

`bindToMockMvc` 또는 `bindToApplicationContext`를 사용할 때, Spring Security의 필터 체인은 서블릿 컨테이너 없이 Mock 환경에서 동작합니다.

```java
@SpringBootTest
@AutoConfigureMockMvc
class SecureBookControllerTest {
    @Autowired
    private WebApplicationContext context;
    
    @Test
    void testWithSpringSecurity() {
        MockMvc mockMvc = MockMvcBuilders
            .webAppContextSetup(context)
            .apply(springSecurity())
            .build();
        
        RestTestClient client = RestTestClient.bindTo(mockMvc).build();
        
        client.post().uri("/api/books")
            .contentType(MediaType.APPLICATION_JSON)
            .body(new Book(null, "Secure Book"))
            .exchange()
            .expectStatus().isForbidden(); // CSRF 토큰 없음
    }
}
```

**설정 자동화:** `SecurityMockMvcConfigurers.springSecurity()`를 MockMvc 빌더에 적용함으로써, 스프링 시큐리티 필터가 테스트 파이프라인에 통합됩니다.

**CSRF 토큰 처리:** RestTestClient는 Mock 환경에서 CSRF 토큰을 자동으로 주입받지 못할 수 있습니다. MockMvc의 RequestPostProcessor 기능을 활용하거나, 테스트 설정에서 CSRF를 비활성화할 수 있습니다. 가장 권장되는 방식은 bindTo 시점에 보안 설정이 완료된 MockMvc 인스턴스를 주입하는 것입니다.

## Live Server: 실제 서버에서의 인증

실제 서버를 대상으로 하는 테스트에서는 모의 객체를 사용할 수 없습니다. 실제 클라이언트처럼 행동해야 합니다.

### HTTP Basic Auth

```java
client.get().uri("/secure")
    .header("Authorization", "Basic " + 
        Base64.getEncoder().encodeToString("user:pass".getBytes()))
    .exchange()
    .expectStatus().isOk();
```

### OAuth2 및 JWT

OAuth2 리소스 서버를 테스트할 때는 유효한 JWT 토큰이 필요합니다.

**Mocking 전략:** 테스트용 프로파일에서 `JwtDecoder`를 Mock Bean으로 등록하여, 서명 검증 없이 임의의 토큰을 허용하도록 설정할 수 있습니다.

**Real Token 전략:** 통합 테스트 전용 계정으로 인증 서버(Keycloak 등, Testcontainers 활용 가능)에서 토큰을 발급받은 후, RestTestClient의 기본 헤더로 설정하여 테스트를 수행합니다. 이는 프로덕션 환경과 가장 유사한 검증 방식입니다.

### CSRF 핸드셰이크 시뮬레이션

`bindToServer` 모드에서 CSRF가 활성화된 엔드포인트(POST/PUT/DELETE)를 테스트하려면 브라우저와 동일한 핸드셰이크 과정이 필요합니다.

1. **GET 요청:** 안전한 메서드로 접근하여 서버로부터 CSRF 토큰(쿠키 또는 헤더)을 수신
2. **토큰 추출:** 응답 쿠키(XSRF-TOKEN)를 추출
3. **POST 요청:** 추출한 토큰을 요청 헤더(X-XSRF-TOKEN)에 포함하여 전송

RestTestClient는 상태를 저장하지 않으므로(Stateless), 이러한 쿠키-헤더 매핑 로직을 테스트 유틸리티로 구현하여 재사용하는 것이 좋습니다.

---

# 데이터 검증: AssertJ와 JSONPath

테스트의 신뢰도는 검증(Assertion)의 정밀도에 달려 있습니다. RestTestClient는 다양한 검증 도구와의 통합을 지원합니다.

## AssertJ 통합

Spring Boot 개발자들에게 익숙한 AssertJ는 가독성 높은 에러 메시지와 풍부한 API를 제공합니다.

```java
client.get().uri("/api/books/1")
    .exchange()
    .expectBody(Book.class)
    .value(book -> {
        assertThat(book.getTitle()).startsWith("Spring");
        assertThat(book.getPages()).isGreaterThan(100);
        assertThat(book.getTags()).contains("java", "spring");
    });
```

단순한 값 비교를 넘어, 컬렉션의 포함 여부, 문자열 패턴 매칭 등 복잡한 비즈니스 규칙 검증에 탁월합니다.

## JSONPath 활용

구조적인 JSON 데이터를 검증할 때, 객체로 매핑하지 않고 원본 JSON 구조를 직접 검사해야 할 때가 있습니다.

```java
client.get().uri("/api/books/1")
    .exchange()
    .expectStatus().isOk()
    .expectBody()
    .jsonPath("$.title").isEqualTo("Spring Boot 4")
    .jsonPath("$.author.name").isEqualTo("Expert")
    .jsonPath("$.tags[0]").isEqualTo("spring");
```

`$.store.book.title`과 같은 경로 표현식을 사용하여 특정 필드의 존재 여부나 값을 검증합니다. 응답 스키마가 변경되었을 때 객체 매핑 에러 없이 유연하게 대처할 수 있습니다.

## JsonUnit: 정교한 JSON 비교

JSON 간의 비교가 필요한 경우, JsonUnit 라이브러리와 통합하여 엄격하거나 유연한 비교를 수행할 수 있습니다. 예를 들어, 배열의 순서를 무시하거나 특정 필드(ID, Timestamp)를 무시하고 비교하는 것이 가능합니다.

---

# Spring Boot 4 생태계 변화

Spring Boot 4의 새로운 기능들은 테스팅 환경에도 지대한 영향을 미칩니다.

## Context Pausing: 메모리 효율 혁신

Spring Framework 7의 가장 혁신적인 테스팅 기능 중 하나는 컨텍스트 캐싱 메커니즘의 개선입니다.

기존에는 테스트 슈트가 실행되는 동안 로딩된 모든 ApplicationContext가 메모리에 상주하며, 백그라운드 스레드나 커넥션 풀을 점유하고 있었습니다. 이는 대규모 프로젝트에서 OOM(Out of Memory)을 유발하거나 테스트 속도를 저하시키는 원인이었습니다.

Spring 7부터는 현재 실행 중이지 않은 테스트의 컨텍스트를 '일시 중지(Pause)'하고, 필요시 '재개(Resume)'합니다. 이는 RestTestClient를 사용하는 수많은 통합 테스트들이 서로 간섭 없이, 그리고 적은 리소스로 수행될 수 있음을 의미합니다.

## Virtual Threads: 블로킹 I/O의 혁명

Java 21의 가상 스레드는 블로킹 I/O 처리의 패러다임을 바꿨습니다. RestTestClient가 `bindToServer` 모드에서 동작할 때, 내부 RestClient는 가상 스레드 친화적인 방식으로 동작하도록 최적화되어 있습니다.

이는 다수의 외부 API 호출을 포함하는 통합 테스트 시나리오에서 스레드 고갈 없이 높은 처리량을 낼 수 있게 하며, 테스트 실행 속도 단축에도 기여합니다.

## AOT 컴파일 지원

Spring Boot 4는 GraalVM 네이티브 이미지를 위한 AOT 지원을 강화했습니다. 테스팅 환경에서도 AOT 처리를 고려한 설정이 필요하며, RestTestClient는 이러한 AOT 모드에서도 문제없이 동작하도록 설계되어, 네이티브 이미지 호환성 테스트(`@DisabledInAotMode` 등)를 용이하게 합니다.

---

# 도구 비교: 무엇을 선택할 것인가?

프로젝트 성격에 따라 올바른 도구를 선택하는 것이 중요합니다.

| 특성 | RestTestClient | WebTestClient | MockMvc | TestRestTemplate |
|-----|---------------|---------------|---------|------------------|
| 기반 스택 | Servlet (Sync) | WebFlux (Async) | Servlet (Mock) | Servlet (Sync) |
| API 스타일 | Fluent (Builder) | Fluent (Builder) | Static Builder | Imperative |
| Mock 지원 | 완벽 지원 (Delegate) | 지원 (Adapter 필요) | Native | 지원 안 함 |
| Live Server | 완벽 지원 | 완벽 지원 | 지원 안 함 | Native |
| 리액티브 의존성 | 불필요 | 필수 (Reactor) | 불필요 | 불필요 |
| 미래 전망 | 표준 (Standard) | 표준 (Reactive) | 백엔드 엔진화 | Deprecated |

**시사점:** RestTestClient는 MockMvc와 TestRestTemplate의 장점을 모두 흡수하고 단점을 보완한 '완전체'에 가깝습니다. 특히 리액티브 의존성 없이 Fluent API를 사용할 수 있다는 점은 기존 Spring MVC 프로젝트들에게 가장 큰 매력 포인트입니다.

---

# 전략적 제언

본 분석을 바탕으로 다음 전략을 제시합니다.

## 1. 신규 프로젝트: 무조건 RestTestClient

Spring Boot 4 기반의 신규 프로젝트는 예외 없이 RestTestClient를 기본 테스팅 클라이언트로 채택해야 합니다. 단위 테스트(`bindToController`)부터 E2E 테스트(`bindToServer`)까지 단일 API로 통일함으로써 팀 내 기술 부채를 예방할 수 있습니다.

## 2. 기존 프로젝트: 점진적 마이그레이션

`@AutoConfigureTestRestTemplate`을 활용하여 안정성을 유지하되, 신규 기능 개발 및 리팩토링 시점마다 RestTestClient로 전환하는 점진적 전략을 수립해야 합니다.

## 3. 바인딩 전략의 이원화

- **속도가 중요한 CI 파이프라인의 PR 단계:** `bindToController` 및 `bindToMockMvc`를 주력으로 사용
- **배포 전 최종 검증 단계(Nightly Build):** `bindToServer`를 활용한 E2E 테스트로 효율성과 신뢰성의 균형 확보

## 4. 보안 테스트 강화

`bindToMockMvc`와 `SecurityMockMvcConfigurers`의 조합을 통해, 단순한 기능 구현 여부를 넘어 보안 필터 체인의 빈틈없는 검증을 자동화해야 합니다.

---

# 마무리하며

RestTestClient는 단순한 도구의 교체가 아닙니다. 테스트 코드를 '검증해야 할 대상'에서 '살아있는 문서'로 변화시키는 촉매제입니다.

**핵심 요약:**

1. **통합된 인터페이스:** Mock과 Live Server를 단일 API로 처리
2. **Fluent API:** 선언적이고 가독성 높은 테스트 코드
3. **바인딩 전략:** 시나리오에 맞는 실행 엔진 선택
4. **Spring Boot 4 최적화:** Context Pausing, Virtual Threads, AOT 지원
5. **마이그레이션:** TestRestTemplate에서 점진적 전환

오랜만에 테스팅 코드를 작성하거나, Spring Boot 4 마이그레이션을 준비 중이라면 이 글을 북마크해두세요. 필요할 때 다시 읽어보면 도움이 될 겁니다! 🚀

---

# 부록: 빠른 참조표

## 주요 AssertJ 검증 패턴

| 검증 대상 | 기존 방식 (TestRestTemplate + AssertJ) | 권장 방식 (RestTestClient) |
|---------|---------------------------------------|---------------------------|
| 상태 코드 | `assertThat(resp.getStatusCode()).isEqualTo(HttpStatus.OK)` | `.expectStatus().isOk()` |
| 헤더 존재 | `assertThat(resp.getHeaders().containsKey("X-Token")).isTrue()` | `.expectHeader().exists("X-Token")` |
| JSON 필드 | `assertThat(resp.getBody().get("name")).isEqualTo("Test")` | `.expectBody().jsonPath("$.name").isEqualTo("Test")` |
| 객체 동등성 | `assertThat(resp.getBody()).usingRecursiveComparison().isEqualTo(dto)` | `.expectBody(Dto.class).isEqualTo(dto)` |

## Spring Boot 4 마이그레이션 체크리스트

- [ ] Java 버전이 17 이상인지 확인 (권장: Java 21 LTS)
- [ ] `spring-boot-starter-parent` 버전을 4.0.0 이상으로 변경
- [ ] `spring-boot-resttestclient` 의존성 추가 여부 확인 (필요 시)
- [ ] 기존 TestRestTemplate 테스트에 `@AutoConfigureTestRestTemplate` 추가
- [ ] `javax.*` 패키지 import를 `jakarta.*`로 일괄 변경 (Servlet 6.1, JPA 3.2 대응)
- [ ] 주요 통합 테스트 시나리오 하나를 선정하여 RestTestClient로 변환 후 성능 및 가독성 비교

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