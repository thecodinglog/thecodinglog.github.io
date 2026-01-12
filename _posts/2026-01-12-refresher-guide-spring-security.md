---
layout: post
title: Spring Security - Your Memory Refresher Guide 🔐
description: Spring Security - Your Memory Refresher Guide 🔐
date:   2026-01-12 11:16:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	Spring Security
---

Ever opened up a Spring Security config after a few months away and thought "Wait, what does this even do?" You're not alone. All those filter chains and security matchers can get jumbled up in your head pretty quickly. This guide will help you rebuild that mental model with Spring Security 7's core architecture.

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

## The Foundation: Understanding Servlet Filters

Before diving into Spring Security, let's talk about servlet filters—they're the foundation everything else builds on.

When a client sends an HTTP request, the servlet container creates a `FilterChain` containing `Filter` instances and a final `Servlet` (in Spring MVC, that's the `DispatcherServlet`).

Filters can do two main things:
- Block the request from reaching downstream filters or the servlet (usually by writing the response directly)
- Modify the request or response before passing it along

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response,
                     FilterChain chain) throws IOException, ServletException {
    // Do something before
    chain.doFilter(request, response); // Pass it along
    // Do something after
}
```

Here's the kicker: **filter order matters. A lot.** Filters execute sequentially, and each one only affects what comes after it.

## DelegatingFilterProxy: Bridging Two Worlds

Here's a problem: servlet containers register filters using their own standards, but they don't know anything about Spring beans.

Enter `DelegatingFilterProxy`. It registers with the servlet container but delegates all the actual work to a Spring bean that implements `Filter`.

```java
public void doFilter(ServletRequest request, ServletResponse response,
                     FilterChain chain) {
    Filter delegate = getFilterBean(someBeanName); // Get the Spring bean
    delegate.doFilter(request, response); // Delegate the work
}
```

Bonus: this allows lazy loading of filter beans. The container needs filters registered early, but Spring beans load later via `ContextLoaderListener`.

## FilterChainProxy: Where the Magic Happens

All of Spring Security's servlet support lives inside `FilterChainProxy`. It's a special filter that can delegate to multiple filter instances through `SecurityFilterChain`. Since `FilterChainProxy` is a bean, it's typically wrapped in a `DelegatingFilterProxy`.

Why `FilterChainProxy` rocks:
- **Single entry point** for all Spring Security servlet support (perfect spot for debugging breakpoints)
- **Clears the SecurityContext** to prevent memory leaks
- **Applies HttpFirewall** to protect against certain attacks
- **Flexible matching** based on anything in the `HttpServletRequest`, not just the URL

## SecurityFilterChain: The Actual Filters

`SecurityFilterChain` determines which Spring Security filters should execute for a given request.

You can have multiple `SecurityFilterChain` instances in one application. `FilterChainProxy` picks the right one, and here's the important part: **only the first matching chain executes.**

Example flow:
- Request to `/api/messages/` → matches `/api/**` pattern in `SecurityFilterChain0` → executes that chain only
- Request to `/messages/` → doesn't match `SecurityFilterChain0` → tries next chains

Each `SecurityFilterChain` is independently configurable with different numbers of filters. You can even have a chain with zero filters if you want Spring Security to ignore certain requests.

## Security Filters: Order Matters

Security filters execute in a specific order. Authentication filters must run before authorization filters, for instance.

Check out this configuration:

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

This creates the following filter order:

1. `CsrfFilter` - CSRF attack protection
2. `BasicAuthenticationFilter` - HTTP Basic authentication
3. `UsernamePasswordAuthenticationFilter` - Form login authentication
4. `AuthorizationFilter` - Authorization

## Getting Started: The Basics

The most basic Spring Security configuration looks like this:

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

This simple config gives you:
- Authentication required for all URLs
- Auto-generated login form
- Form-based authentication
- Logout support
- CSRF attack prevention
- Session fixation protection
- Security header integration (HSTS, X-Content-Type-Options, etc.)
- Servlet API method integration

Not bad for a few lines of code!

## HttpSecurity: Fine-Grained Control

That basic config actually creates this `SecurityFilterChain` behind the scenes:

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

This setup:
- Requires authentication for every request
- Enables form login
- Enables HTTP Basic authentication

## Multiple HttpSecurity: Different Rules for Different Areas

Real applications often need different security configurations for different parts. Just register multiple `SecurityFilterChain` beans:

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
    @Order(1)  // Higher priority
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")  // Only applies to /api/**
            .authorizeHttpRequests((authorize) -> authorize
                .anyRequest().hasRole("ADMIN")
            )
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }

    @Bean  // No @Order = lowest priority
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

## securityMatcher vs requestMatchers: Know the Difference

This trips people up all the time:

- `http.securityMatcher()`: Determines which requests this entire `SecurityFilterChain` applies to
- `requestMatchers()`: Determines which requests individual authorization rules apply to within the chain

```java
@Bean
public SecurityFilterChain securedFilterChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/secured/**")  // Chain applies to /secured/** only
        .authorizeHttpRequests((authorize) -> authorize
            .requestMatchers("/secured/user").hasRole("USER")    // Specific rule
            .requestMatchers("/secured/admin").hasRole("ADMIN")  // Specific rule
            .anyRequest().authenticated()
        )
        .httpBasic(Customizer.withDefaults())
        .formLogin(Customizer.withDefaults());
    return http.build();
}
```

**Critical point:** If you specify a `securityMatcher`, only matching requests are protected. Non-matching requests won't be protected by Spring Security at all! That's why it's recommended to have a default chain without a `securityMatcher`.

## SecurityFilterChain Endpoints: A Gotcha

Endpoints provided by the filter chain (like `/login`, `/logout`) aren't automatically affected by `securityMatcher`:

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
            .loginPage("/secured/login")           // Custom path
            .loginProcessingUrl("/secured/login")  // Custom path
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
            .anyRequest().denyAll()  // Deny everything else
        );
    return http.build();
}
```

## Real-World Example: A Banking System

Let's look at a more complex, realistic example:

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
    @Order(1)  // Highest priority
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
    @Order(2)  // Second priority
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

    @Bean  // Default chain (lowest priority)
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

How this works:

1. **Approval paths** (`@Order(1)`): `/accounts/approvals/**`, `/loans/approvals/**`, `/credit-cards/approvals/**` require ADMIN role and use HTTP Basic auth

2. **Banking paths** (`@Order(2)`): For `/accounts/**`, `/loans/**`, `/credit-cards/**`, `/balances/**`:
   - `/balances/**` requires `VIEW_BALANCE` role
   - Everything else requires `USER` role
   - Requests with `/approvals/` already matched the first chain, so they won't hit this one

3. **Default paths** (lowest priority):
   - `/`, `/user-login`, `/user-logout`, `/notices`, `/contact`, `/register` are publicly accessible
   - Everything else requires authentication
   - Uses form login

## Wrapping Up

Spring Security essentials in a nutshell:

1. **Filter-based architecture**: Built on servlet filters
2. **DelegatingFilterProxy**: Bridges servlet container and Spring
3. **FilterChainProxy**: Routes requests to the right SecurityFilterChain
4. **SecurityFilterChain**: Collection of actual security filters
5. **Priority**: Use `@Order` to control chain execution order
6. **Matcher distinction**: `securityMatcher` (chain scope) vs `requestMatchers` (individual rules)

Bookmark this page for the next time you need to dust off your Spring Security knowledge. You'll thank yourself later! 🚀

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