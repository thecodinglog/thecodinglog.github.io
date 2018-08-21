---
layout: post
title: Spring Security로 Security 서비스 구축하기 3
description: Spring Security voter 구현하기
date:   2018-08-15 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---

앞선 포스트에서는 요청한 View가 사용자가 가진 View Type SecuredObject 중에 존재하는지 찾아봐서 접근허가 여부를 결정짓는 ViewVoter를 구현하였다. 이 구현체에 NPE을 대비한 코드를 조금 더 추가해서 완성도를 높혔다.

```java
package cothe.security.access.vote;

public class ViewVoter implements AccessDecisionVoter<Object> {
    private RoleProvider roleProvider;

    public ViewVoter(RoleProvider roleProvider) {
        this.roleProvider = roleProvider;
    }

    @Override
    public boolean supports(ConfigAttribute attribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }

    @Override
    public int vote(Authentication authentication, Object object, Collection<ConfigAttribute> attributes) {
        Assert.notNull(this.roleProvider, "There is no role provider.");

        String targetView = (String) object;

        if (!authentication.isAuthenticated()) {
            return ACCESS_DENIED;
        }

        for (GrantedAuthority authority : authentication.getAuthorities()) {
            Role role = roleProvider.getRole(authority.getAuthority());

            if (role == null)
                continue;

            if (Optional.ofNullable(role.getPermissions())
                    .map(permissions -> permissions.stream()
                            .filter(permission -> permission.getSecuredObject().getSecuredObjectType() == SecuredObjectType.VIEW)
                            .anyMatch(permission -> permission.getSecuredObject().getSecuredObjectId().equals(targetView))
                    ).orElse(false)) {
                return ACCESS_GRANTED;
            }
        }
        return ACCESS_DENIED;
    }
}

```

이를 검증하기 위한 테스트도 추가했다.

```java
package cothe.security.access.vote;

@RunWith(SpringRunner.class)
@WithMockSecuredUser(username = "admin", name = "admin", roles = "default_view_permission")
public class ViewVoterTest {
    ...

    @Test
    public void permission이null이면(){
        ViewVoter nullPermissionViewVoter = new ViewVoter(new MockRoleProvider(null));

        String targetView = "default_object";
        int voteResult = nullPermissionViewVoter.vote(authentication, targetView, null);
        assertTrue(voteResult < 0);
    }

    ...
}
```
1.SecurityContextHolder에서 Authentication 개체를 가져옵니다.
2.SecurityMetadataSource에 대한 보안 오브젝트 요청을 조사하여 요청이 보안 호출 또는 공개 호출과 관련되는지 판별
3.보안 호출의 경우 (보안 오브젝트 호출에 대한 ConfigAttributes 목록이 있음)
3-1.Authentication.isAuthenticated ()가 false를 반환하거나 alwaysReauthenticate가 true이면 구성된 AuthenticationManager에 대해 요청을 인증
3-2.인증 될 때 SecurityContextHolder의 Authentication 개체를 반환 된 값으로 바꿉
3-3.구성된 RunAsManager를 통해 임의의 Run-as 대체를 수행
3-4.콘트롤을 구체적인 서브 클래스로 다시 넘겨 주면 실제로 객체를 수행하게됩니다.InterceptorStatusToken이 반환되어 서브 클래스가 객체의 실행을 끝내고 나서 finally 절을 사용하면 finallyInvocation (InterceptorStatusToken)을 사용하여 AbstractSecurityInterceptor를 다시 호출하고 올바르게 정리할 수 있습니다.
3-5.구체적인 서브 클래스는 afterInvocation (InterceptorStatusToken, Object) 메소드를 통해 AbstractSecurityInterceptor를 다시 호출합니다.
3-6.RunAsManager가 Authentication 개체를 바꾼 경우 SecurityManager를 호출 한 후에 있던 개체에 SecurityContextHolder를 반환합니다.
3-7.AfterInvocationManager가 정의 된 경우 호출 관리자에게 호출하고 호출자에게 리턴 될 예정으로 인해 오브젝트를 대체하도록 허용하십시오.
4.공용 인 호출의 경우 (보안 오브젝트 호출에 대한 ConfigAttributes가 없음).
4-1.전술 한 바와 같이, 구체적인 서브 클래스는 InterceptorStatusToken으로 리턴 될 것이고, 보안 객체가 실행 된 후에 AbstractSecurityInterceptor로 다시 제시된다. AbstractSecurityInterceptor는 afterInvocation (InterceptorStatusToken, Object)이 호출 될 때 더 이상의 조치를 취하지 않습니다.
5.컨트롤은 다시 호출자에게 반환되어야하는 Object와 함께 구체적인 하위 클래스로 반환됩니다. 서브 클래스는 그 결과 나 예외를 원 호출자에게 리턴 할 것이다.


AffirmativeBased의 부모 클래스인 AbstractAccessDecisionManager 의 supports는 AccessDecisionManager가 가지고 있는 Voter들의 supports를 하나씩 돌려봐서 하나라도 true가 리턴되면 전체 true로 리턴함



Web Security
DelegatingFilterProxy 라는 Filter 를 web.xml 에 등록시킨다.
DelegatingFilterProxy 는 자체적으로 비즈니스 로직을 가지지 않고 스프링 애플리케이션 컨텍스트에서 가져온 필터들의 메소드를 대신 실행시켜주는 역할을 한다. 스프링 애플리케이션 컨텍스트에 등록된 빈들은 javax.servlet.Filter 를 구현해야 한다.
스프링 시큐리티의 웹 인프라는 FilterChainProxy 인스턴스에 위임해야만 사용할 수 있다. 스큐리티 필터들을 독립적으로 사용하면 안된다. 이론적으로는 web.xml 에 등록해서 쓸 수 있지만 번거롭고 복잡하므로 그렇게 하지 마라.

