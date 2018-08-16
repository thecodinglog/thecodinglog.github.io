---
layout: post
title: Spring Security로 Security 서비스 구축하기 2
description: Spring Security voter 구현하기
date:   2018-08-15 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---
앞선 포스트에서는 SpringSecurity에서 인증을 할 수 있도록 관련 클래스를 정의하고 테스트를 해보았다. 이번 포스트는 사용자가 가지고 있는 **Role** 을 기반으로 접근 권한이 있는지 없는지 확인하는 `Voter` 를 만들어 테스트할 것이다. 

# 사용자 오브젝트가 권한을 가지도록 변경

`User` Class에 `roles` 필드 추가
```java
package cothe.security.core.domain;

@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    @Id
    private String userId;
    private String password;
    private boolean enabled;
    private Set<Role> roles;
}
```
Role Class 추가

```java
package cothe.security.core.domain;

@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Role {
    private String roleId;
    private String roleName;
    private Role parentRole;
}
```

인증테스트의 `setUp` 부분에서 `Role` 오브젝트를 저장하도록 수정

`AuthenticateTest.java`

```java
    ...

    @Before
    public void setUp() throws Exception {

        userRepository = new MockUserRepository();
        userRepository.save(
                new User("cothe", "pass", true,
                        Stream.of(Role.builder()
                                .roleId("admin")
                                .roleName("관리자").parentRole(null)
                                .build()).collect(Collectors.toSet())
                ));

        userDetailsService = new MockUserDetailsService(userRepository);

        authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService);
        authenticationProvider.setPasswordEncoder(NoOpPasswordEncoder.getInstance());
        authenticationProviders = Stream.of(
                authenticationProvider
        ).collect(Collectors.toList());

        providerManager = new ProviderManager(authenticationProviders);
    }

    ...
```
`User`는 `Role` 들을 가지고 있다. 그래서 `Set<Role>`형의 `roles`를 추가하여 앞선 테스트를 다시 실행해 보면 예외가 발생한다. JPA에서는 다른 객체를 참조하는(String과 같은 일부 네이티브 타입은 제외) 필드는 어떤 관계(다대일, 일대다 ...)인지 반드시 명시하게 되어있기 때문이다.

# Role을 Entity 클래스로 설정

### Role을 Entity로

```java
package cothe.security.core.domain;

@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Role {
    @Id
    private String roleId;
    private String roleName;

    @ManyToOne
    @JoinColumn(name = "parent_user_id")
    private Role parentRole;
}
```

```java
private Role parentRole;
```
에서 `@ManyToOne` 으로 관계를 설정했다. 자식 `Role` 은 여러개(Many)이고 부보는 하나(One)이기 때문이다.


### User와 Role의 관계 설정

```java
package cothe.security.core.domain;

@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    @Id
    private String userId;
    private String password;
    private boolean enabled;

    @OneToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "user_role"
            , joinColumns = @JoinColumn(name = "user_id")
            , inverseJoinColumns = @JoinColumn(name = "role_id"))    
    private Set<Role> roles;
}
```

한 `User`(One)는 여러 `Role`(Many)을 가지고 있기 때문에 `@OneToMany`로 지정했다. 사실 `User`와 `Role`은 다대다 관계를 맺고 있다. 객체와 객체는 다대다 관계를 표현하기 쉽지만 RDB는 중간에 _JoinTable_ 을 사용해서 그 관계를 정의한다. fetch 타입은 `EAGER`로 설정했다. `User`가 로드되면 `Role`정보도 같이 로드 되도록 하기 위함이다. 디폴트 값은 `LAZY`인데 `User`를 로드하면 대부분 `Role`에도 접근하므로 효율성 향상을 위해 이렇게 설정했다.

### 테스트
테스트가 정상적으로 통과됐고 DB에는 테이블 3개가 생성된 것을 볼 수 있다.
![](/assets/2018-07-31-spring-security-service/2018-07-31-spring-security-service_124824.png)

# 권한 구조 정의

엔터프라이즈 애플리케이션은 몇 가지 특징이 있다.
* 기본적으로 한 화면이 매우 복잡한 구조를 가지고 권한 자체도 화면 안에 오브젝트마다 정의될 필요가 있다. 예를 들어 어떤 화면을 열어서 조회기능은 할 수 있지만 저장을 할 수 없도록 버튼 자체를 비활성화하거나 없애야 할 수도 있다.
* 화면 수도 매우 많아 권한 관리가 적절히 되지 않으면 사용자를 관리하는데 상당히 복잡한 문제를 격을 수 있다. A 팀에 속한 유저는 기본적으로 A역할을 가지는 정책이 있다고 하자. 그런데 어느 특정 유저만 특별히 다른 권한을 줘야 할 수도 있다. 이를 위해 A역할허가+특수허가를 넣고 A-1 역할이라는 것을 만든다. 새로 만든 권한이 정말 잘 관리가 되어야 하는데 권한관리 담당자가 바뀌어 인수인계가 잘 안 되면 A역할이 있는지도 모르고 새로운 유저에게 모두 A-1 역할을 줄 수도 있다. 다르게는 A-1과 같은 역할을 계속해서 만들어 낼 수도 있다.
* 한 사람이 여러 가지 역할을 가질 수 있는데 역할별로 허가(권한)가 상충할 수 있다. 예를 들어 A 역할에는 주문화면에 접근 권한이 있지만 B 역할에는 없을 수 있다. 이럴 때 어떤 권한 부여 정책을 줄지 결정해야 하는데 여기서는 긍정적 접근방법(A 롤에 권한이 있으면 다른 롤은 무시함)으로 진행할 것이다.
* 여러 화면으로 이리저리 흘러 다니는 구조보다는 한 화면에서 조회, 저장 등 많은 트랜잭션을 일으킨다. 그래서 AJAX 통신이 빈번히 일어나는데 이 ajax 요청에 대하여 권한 체크는 반드시 있어야 한다. 저장을 못 하도록 버튼을 숨기더라도 UI에서 얼마든지 URL을 조작하여 저장요청을 만들 수 있기 때문이다.
* 때에 따라서는 요청받은 데이터 자체(쿼리스트링)를 검증해야 할 수도 있다. 같은 URL로 저장요청을 해도 특정 파라미터의 값은 받아들이지 않도록 해야 할 수 있다. 예를 들면 주문에 지급방법이라는 파라미터 paymentMethod 이라는 것이 있을 때 특정 사용자는 외상을 할 수 **없다고** 해보자. 다시 말해 URL이 `https://salesmarket/order?productNo=1234&paymentMethod=create` 와 유사하다고 했을 때 이 요청에 `paymentMethod=create` 는 받아들이지 않도록 해야 한다는 의미이다.
* 마스터 코드 데이터도 권한에 따라 조회되는 값이 달라야 할 수 있다. 같은 화면을 권한이 다른 많은 사람이 사용하고 있기 때문이다. 

뭐 한마디로 이야기하면 권한 관리하기가 매우 복잡하고 힘들다는 말이다. 권한 구조를 말로 표현해보자.

**사용자(User)** 는 다양한 **역할(Role)** 을 가질 수 있다. 예를 들면 전체 서비스를 관리하는 관리자 역할, 팀별 고유 역할, 일반 외부 사용자 역할 등이 있을 수 있다. 각 역할은 무언가를 할 수 있는 **허가(Permission)** 들을 가진다. 각 허가는 보호되어야 할 **오브젝트(SecuredObject)** 와 일대일 대응 되고 **구체적으로 어떤 허가를 가지고 있는지 명세** 를 가지고 있다.

# SecuredObject 클래스 정의
보호되어야 할 오브젝트는 URL일 수도 있고, 메소드 일 수도 있고, 클래스 자체일 수도 있다. 조금 더 거시적인 관점에서 보면 View, Service 가 보호해야 할 대상이라고 표현할 수도 있다.  
View는 로그인 화면, 주문 화면, 관리자 화면 등 사용자와 직접 상호작용 할 수 있는 사용자 인터페이스 화면이라고 할 수 있겠다.
Service는 주문 데이터를 요청하는 Order Service, 생산 Service 등 도메인에서 일어날 수 있는 이벤트 처리기이자 발생기라고 볼 수 있겠다.

### Secured Object의 타입

```java
package cothe.security.core.domain;

public enum SecuredObjectType
{
    SERVICE,
    VIEW
}
```
### Secured Object 클래스

단순히 id, 이름, 타입정보를 가진다.

```java
package cothe.security.core.domain;

@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class SecuredObject {
    @Id
    private String SecuredObjectId;
    private String SecuredObjectName;

    @Enumerated(value = EnumType.STRING)
    private SecuredObjectType securedObjectType;
}
```


# Permission 클래스 정의
permission filed는 연결된 오브젝트에 대한 권한정보 스크립트를 담을 예정이다.
```java
package cothe.security.core.domain;

@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Permission {
    @Id
    private String permissionId;
    private String permissionName;
    private String permission;
    @ManyToOne
    @JoinColumn(name = "secured_object_id")
    private SecuredObject securedObject;

}
```
# 테스트 작성
```java
package cothe.security.access;

public class AccessDecisionTest {
    private User user1;
    private SecuredObject view1;
    private Permission view1Permission1;
    private Role roleAdmin;


    @Before
    public void setUp() throws Exception {

        view1 = SecuredObject.builder()
                .SecuredObjectId("view1")
                .SecuredObjectName("view1")
                .securedObjectType(SecuredObjectType.VIEW)
                .build();

        view1Permission1 = Permission.builder()
                .permissionId("view1Permission1")
                .permissionName("view1Permission1")
                .securedObject(view1)
                .build();

        roleAdmin = Role.builder()
                .roleId("role_admin")
                .roleName("관리자")
                .permissions(
                        Stream.of(view1Permission1).collect(Collectors.toSet())
                ).build();

        user1 = User.builder()
                .userId("cothe")
                .password("pass")
                .enabled(true)
                .roles(
                        Stream.of(roleAdmin).collect(Collectors.toSet())
                ).build();

    }

    @Test
    public void hasRole() {
        //given

        //when then
        assertTrue(user1.getRoles().stream().anyMatch(role -> role.getRoleId().equals("role_admin")));
    }

    @Test
    public void View에접근권한있는지테스트(){
        String targetViewId = "view1";

        assertTrue(user1.getRoles().stream().anyMatch(
                role -> role.getPermissions().stream().anyMatch(
                        permission -> permission.getSecuredObject().getSecuredObjectId().equals(targetViewId)
                )
        ));
    }
    @Test
    public void View에접근권한없는지테스트(){
        String targetViewId = "view2";

        assertFalse(user1.getRoles().stream().anyMatch(
                role -> role.getPermissions().stream().anyMatch(
                        permission -> permission.getSecuredObject().getSecuredObjectId().equals(targetViewId)
                )
        ));
    }
}

```

### 테스트 결과
![](/assets/2018-07-31-spring-security-service-2/2018-07-31-spring-security-service-2_174251.png)

# Spring Security 접근 권한관련 클래스 구조
앞선 실습에서 `User` 도메인 객체가 `Role` 들을 가지고 있도록 했다. 이 객체로 인증을 하기 위해서 Spring Security의 `User` 클래스로 랩핑을 했는데 그 과정에서 `Set<Role>` 을 `Set<SimpleGrantedAuthority>` 로 전환하는 과정이 있었다. 이 정보를 `AccessDecisionManager` 가 읽어서 권한 체크를 한다. 스프링 시큐리티에서 접근 권한 체크를 하는 일련의 과정을 만들어보겠다.  

 먼저 스프링 시큐리티의 접근 권한 관련 클래스들의 구조를 보자.

![](/assets/2018-07-31-spring-security-service/2018-07-31-spring-security-service_170744.png)

최상단에 `AccessDecisionManager` 가 있다. 이 인터페이스는 `decide` 메소드로 권한 체크를 하게 되는데 권한이 없으면 `AccessException` 을 던지게 되어있다.

```java
package org.springframework.security.access;

public interface AccessDecisionManager {
	void decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
			InsufficientAuthenticationException;
	boolean supports(ConfigAttribute attribute);
	boolean supports(Class<?> clazz);
}
```

`AccessDecisionManager` 를 구현한 추상클래스 `AbstractAccessDecisionManager` 는 내부에 `AccessDecisionVoter` 인터페이스 리스트를 저장할 수 있도록 필드가 있다. 이 리스트는 `AccessDecisionManager` 가 생성될 때 초기화되어야 한다. 
`AccessDecisionVoter` 인터페이스는 접근 권한을 실제로 확인하는 vote 메소드를 가지고 있는데 권한 체크 후 `int` 형으로 결과를 돌려준다.

리턴값 | 의미 
---------|----------
 1 | 권한 있음
 0 | 기권
 -1 | 권한 없음


이 `AccessDecisionVoter` 리스트를 어떤 전략으로 체크해서 최종 접근 권한 결정을 하는가에 따라서 구현된 3개 클래스가 `AffirmativeBased`, `ConsensusBased`, `UnanimousBased` 이다. 

 `AffirmativeBased` 는 `AccessDecisionVoter` 중 하나라도 1(권한있음) 이 리턴되면 접근 권한이 있다고 결정한다. `Voter`가 모두 기권인 경우는 별도 설정값에 따라 결정한다.  
 `ConsensusBased` 는 `AccessDecisionVoter` 들이 리턴한 값을 모두 더해서 양수인지 음수인지에 따라서 결정하고 합이 0인 경우와 `AccessDecisionVoter` 들이 모두 기권한 경우는 프로퍼티 설정값에 따라 결정한다.  
 `UnanimousBased` 는 하나라도 -1(권한없음)이 리턴되면 접근 권한이 없다고 결정한다.

# AccessDecisionVoter

실제 접근 권한 체크를 하는 인터페이스이다. `AccessDecisionManager`가 이 인터페이스를 실행한다. `vote` 메소드의 파리미터에는 어떤 값이 와야하는지 보자.

* `authentication` – 권한인증을 시도하는 사용자 인증정보
* `object` – 사용자가 요청한 보호된 오브젝트
* `attributes` – 보호된 오브젝트와 관련된 속정들

```java
package org.springframework.security.access;

public interface AccessDecisionVoter<S> {
	int ACCESS_GRANTED = 1;
	int ACCESS_ABSTAIN = 0;
	int ACCESS_DENIED = -1;
	boolean supports(ConfigAttribute attribute);
	boolean supports(Class<?> clazz);
	int vote(Authentication authentication, S object,
			Collection<ConfigAttribute> attributes);
}
```


# View에 대한 접근권한을 확인하는 Voter 만들기
권한 인증 요청을 받아 `vote` 메소드를 실행하면 넘겨받은 `Authentication`에서 역할을 가져오고, 이 역할을 순회하면서 해당 역할에 접근하고자 하는 보호된 오브젝트에 권한이 있는지 확인한다. 

눈여겨봐야 할 부분은 `authentication`에서 `getAuthorities()` 를 호출하면 `String` 형태의 `roleId` 를 가져올 수 있는데, 이것을 가지고 `Role` 도메인 오브젝트를 가져올 수 있도록 `RolePrivider` Interface를 사용한 것이다. Interface를 사용한 이유는 `Role` domain을 가져오는데 어떻게 가져오는지에 대해서는 `ViewVoter`가 몰라도 되도록 하기 위해서이다. 예를 들어 JPA를 이용해서 DB에서 가져오겠다고 했을 경우 이 Voter는 `JpaRepository` interface에 의존하게 되고 JDBC는 `JdbcTemplte`에 의존하게 된다. 이걸 인터페이스에 의존하게 되면 깔끔하게 정리된다. 지금 당장은 아니지만 앞으로 JPA를 이용해서 `Role` 도메인을 가져올 수 있도록 할 것이기 때문에 JPA용 구현체를 추가로 미리 작성한다.

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
        String targetView = (String) object;

        if (authentication.isAuthenticated()) {
            return ACCESS_DENIED;
        }

        for (GrantedAuthority authority : authentication.getAuthorities()) {
            Role role = roleProvider.getRole(authority.getAuthority());

            if (role == null)
                continue;

            if (role.getPermissions().stream()
                    .filter(permission -> permission.getSecuredObject().getSecuredObjectType()== SecuredObjectType.VIEW)
                    .anyMatch(permission -> permission.getSecuredObject().getSecuredObjectId().equals(targetView))) {
                return ACCESS_GRANTED;
            }
        }
        return ACCESS_DENIED;
    }
}
```
```java
package cothe.security.core.domain.providers;

public interface RoleProvider {
    Role getRole(String roleId);
}
```

```java
package cothe.security.core.domain.providers;

public class RoleProviderJpa implements RoleProvider {
    private RoleRepository roleRepository;

    public RoleProviderJpa(RoleRepository roleRepository) {
        this.roleRepository = roleRepository;
    }

    @Override
    public Role getRole(String roleId) {
        return roleRepository.findById(roleId).orElse(null);
    }
}
```

새로 만든 `Voter`의 테스트를 만들자. SpringSecurity는 테스트를 좀 더 용이하게 하기 위해서 몇 가지 Mock Annotation 을 제공한다. 그것들 중에 `@WithSecurityContext` 를 사용해서 언제든지 `SecurityContext`에서 `Authentication`을 가져올 수 있도록 준비해보자.

`@WithMockSecuredUser` 애노테이션을 새로 정의한다. 이 애노테이션을 쓰면 `SecurityContext`에서 `UserDetails`를 가져올 수 있도록 할 것이다.

```java
package cothe.security.mock;

import org.springframework.security.test.context.support.WithSecurityContext;

@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithMockSecuredUserSecurityContextFactory.class)
public @interface WithMockSecuredUser {
    String username() default "user1";
    String name() default "user one";
    String roles() default "role1,role2";
}
```
이제 `@WithMockSecuredUsed` 애노테이션을 쓸때 `SecurityContext`에 `Authentication`을 만들고 인증정보를 넣어주는 Factory를 만든다.

```java
package cothe.security.mock;

public class WithMockSecuredUserSecurityContextFactory implements WithSecurityContextFactory<WithMockSecuredUser> {
    @Override
    public SecurityContext createSecurityContext(WithMockSecuredUser annotation) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();

        Authentication auth = new UsernamePasswordAuthenticationToken(annotation.username(), "pass",
                Arrays.stream(annotation.roles().
                        split(",")).map(s -> new SimpleGrantedAuthority(s)).collect(Collectors.toSet()));
        context.setAuthentication(auth);
        return context;
    }
}
```
이제 실제 테스트 코드를 작성한다. class level에 `@WithMockSecuredUser` 애노테이션을 붙이고 인증정보를 추가한다. 아래 코드와 같이 인증정보를 넣으면 id가 admin으로 인증이 됐고, `default_view_permission` role을 가진 `Authentication`이 `SecurityContext` 에 세팅될 것이다. 앞서 Factory를 만들때 <kbd>,</kbd> 를 기준으로 여러 role을 정의할 수 있도록 했으므로 참고하기 바란다.

```java
package cothe.security.access.vote;

@RunWith(SpringRunner.class)
@WithMockSecuredUser(username = "admin", name = "admin", roles = "default_view_permission")
public class ViewVoterTest {
    private Set<Permission> permissions;
    private Authentication authentication;
    private ViewVoter viewVoter;

    @Before
    public void setUp() {
        permissions = Stream.of(
                new Permission("default_view_permission", "default_view_permission", null,
                        new SecuredObject("default_object", "default_object", SecuredObjectType.VIEW))
        ).collect(Collectors.toSet());

        authentication = SecurityContextHolder.getContext().getAuthentication();
        viewVoter = new ViewVoter(new MockRoleProvider(permissions));
    }

    @Test
    public void view접근권한있을때(){
        String targetView = "default_object";
        int voteResult = viewVoter.vote(authentication, targetView, null);
        assertTrue(voteResult > 0);
    }

    @Test
    public void view접근권한없을때(){
        String targetView = "view1";
        int voteResult = viewVoter.vote(authentication, targetView, null);
        assertTrue(voteResult < 0);
    }
}
```

메모리에서 `Role` 정보를 임의로 만들어 내는 `MockRolePrivider`를 하나 구현해서 사용했다. `Permission`의 `Set`을 인자로 받던지 디폴트 값을 쓰던지 선택할 수 있다.

```java
package cothe.security.mock;

public class MockRoleProvider implements RoleProvider {
    private Role role;
    private Set<Permission> permissions;

    @Override
    public Role getRole(String roleId) {
        role = new Role(roleId, roleId, null, permissions);

        return role;
    }

    public MockRoleProvider() {
        permissions = Stream.of(
                new Permission("default_view_permission", "default_view_permission", null,
                        new SecuredObject("default_object", "default_object", SecuredObjectType.VIEW))
        ).collect(Collectors.toSet());
    }

    public MockRoleProvider(Set<Permission> permissions) {
        this.permissions = permissions;

    }
}
```

### 테스트 실행 결과
![](/assets/2018-07-31-spring-security-service-2/2018-07-31-spring-security-service-2_161916.png)
