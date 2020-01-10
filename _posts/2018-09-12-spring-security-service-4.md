---
layout: post
title: Spring Security로 Security 서비스 구축하기 4
description: Spring Security voter 구현하기 스프링 시큐리티 스프링시큐리티 access
date:   2018-09-12 15:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
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


# 계층구조 Role

우리가 정의한 `Role`은 부모 `Role`을 가지질 수 있는 계층적인 구조이다. 따라서 부모가 가지고 있는 `Permission` 이 그대로 상속되고, 권한 체크할 때 부모가 가진 `Permission`도 모두 검증해봐야 한다. 그러기 전에 먼저, JPA의 `JpaRepository` interface가 계층적인 구조에서 잘 동작하는지 검증해보자.

`RoleRepositoryTest.java`

```java
package cothe.security.core.repositories;

@RunWith(SpringRunner.class)
@SpringBootTest
public class RoleRepositoryTest {
    @Autowired
    RoleRepository roleRepository;

    @Before
    public void setUp() {
        Role role1 = Role.builder().roleId("role1").build();
        Role role2 = Role.builder().roleId("role2").parentRole(role1).build();
        Role role3 = Role.builder().roleId("role3").parentRole(role2).build();

        roleRepository.save(role1);
        roleRepository.save(role2);
        roleRepository.save(role3);
    }

    @Test
    public void loadHierarchicalRole() {
        Optional<Role> role3_op = roleRepository.findById("role3");
        Role role = role3_op.orElse(null);
        assertEquals(Objects.requireNonNull(role).getParentRole().getParentRole().getRoleId(), "role1");
    }
}
```

![](/assets/2018-09-07-spring-security-service-4/2018-09-07-spring-security-service-4_144646.png)

# Role 계층구조 테스트 작성
계층구조인 `Role`상태에서 `DenialFirstServiceVoterTest`에서 했던 테스트가 잘 동작하는지 테스트해보면 어느 정도 검증일 될 것이다. 편의상 `DenialFirstServiceVoterTest`를 복사해서 `DenialFirstServiceVoterHierarchyTest`라는 이름으로 테스트 클래스를 하나 더 만든다.
여기서 변경해야 할 것은 `User`가 가지고 있는 `Role`과 계층적인 `Role`을 넣어주는 `setUp()`메소드 일부이다.

### 사용자 역할 바꾸기
#### 기존

`@WithMockSecuredUser(username = "admin", name = "admin", roles = "role1,role2,role3")`

#### 변경 후

`@WithMockSecuredUser(username = "admin", name = "admin", roles = "role2")`

### Role 들과의 관계 바꾸기
#### 기존

```java
mockRepositoryRoleProvider.putRole("role2",
        Stream.of(
                new Permission("permission2-1", "permission2-1", serializedPermissionDescriptionOf("permission2-1"),
                        new SecuredObject("object2", "object2", SecuredObjectType.SERVICE)),
                new Permission("permission3", "permission3", serializedPermissionDescriptionOf("permission3"),
                        new SecuredObject("object3", "object3", SecuredObjectType.SERVICE))
        ).collect(Collectors.toSet()));
```

#### 변경 후
```java
mockRepositoryRoleProvider.putRole("role2",
        Stream.of(
                new Permission("permission2-1", "permission2-1", serializedPermissionDescriptionOf("permission2-1"),
                        new SecuredObject("object2", "object2", SecuredObjectType.SERVICE)),
                new Permission("permission3", "permission3", serializedPermissionDescriptionOf("permission3"),
                        new SecuredObject("object3", "object3", SecuredObjectType.SERVICE))
        ).collect(Collectors.toSet()), mockRepositoryRoleProvider.getRole("role1"));
```

이렇게 변경하면 컴파일 에러가 날 것이다. `mockRepositoryRoleProvider`에 정의한 `putRole` 메소드 중에 부모 Role 을 받을 수 있는 생성자를 정의하지 않았기 때문이다.

`MockRepositoryRoleProvider.java` 에 메소드를 아래 메소드를 추가한다.

```java
public void putRole(String roleId, Set<Permission> permissions, Role parentRole){
    roles.put(roleId, new Role(roleId, roleId, parentRole, permissions));
}
```

여기까지 바꾼 뒤 테스트를 돌려본다. 당연하겠지만 **실패**한다. 이제 이 테스트가 통과되도록 `ServiceVoter`를 바꿀 것이다.

# RoleProvider 예외 처리
`RoleProvider` Interface를 보면 `roleId`를 입력받아 `Role`객체를 반환하게 되어있다. 만약 `roleId`에 해당하는 `Role`객체를 만들지 못했을 경우 어떻게 동작해야 할 것인가를 고민해보자.


먼저, 없을 때 null 을 반환하게 하면 사용자의 역할 정보에 오류가 있음에도 모르고 지나갈 수 있다. 사용자가 가지는 `RoleId`는 `String Type`이다. 사용자 관리 UI에서 유효성 검사를 하지 않으면 틀린 Role Id가 들어갈 수 있다.

Role 정보를 찾을 수 없을 때 예외를 발생시키면 예상하지 못한 버그를 막을 수 있다.

### RoleProvider 가 Role 객체를 생성 못 했을 때 발생할 예외 만들기
```java
package cothe.security.core.exceptions;

public class RoleNotFoundException extends RuntimeException {
    public RoleNotFoundException(String message) {
        super(message);
    }
}
```

### RoleProvider interface에 예외 명시하기

```java
package cothe.security.core.domain.providers;

public interface RoleProvider {
    Role getRole(String roleId) throws RoleNotFoundException;
}
```

### RoleProviderJpa 예외 처리
```java
package cothe.security.core.domain.providers;

public class RoleProviderJpa implements RoleProvider {
    private final RoleRepository roleRepository;

    public RoleProviderJpa(RoleRepository roleRepository) {
        this.roleRepository = roleRepository;
    }

    @Override
    public Role getRole(String roleId) {
        if (roleId == null) throw new IllegalArgumentException();

        return roleRepository.findById(roleId).orElseThrow(() -> new RoleNotFoundException(
                String.format("Can't find role id : %s", roleId)));
    }
}
```

### RoleProviderJpa 테스트 작성
```java
package cothe.security.core.domain.providers;

@RunWith(SpringRunner.class)
@SpringBootTest
public class RoleProviderJpaTest {
    @Autowired
    RoleRepository roleRepository;

    RoleProvider roleProvider;

    @Before
    public void setUp(){
        roleProvider = new RoleProviderJpa(roleRepository);

        Role role1 = Role.builder().roleId("role1").build();
        Role role2 = Role.builder().roleId("role2").parentRole(role1).build();
        Role role3 = Role.builder().roleId("role3").parentRole(role2).build();

        roleRepository.save(role1);
        roleRepository.save(role2);
        roleRepository.save(role3);

    }
    @Test
    public void bringRole(){
        Role role = roleProvider.getRole("role3");
        assertEquals(role.getRoleId(), "role3");
    }
    @Test(expected = RoleNotFoundException.class)
    public void bringRoleException(){
        Role role = roleProvider.getRole("role4");

    }

}
```

![](/assets/2018-09-07-spring-security-service-4/2018-09-07-spring-security-service-4_160838.png)

# AbstractServiceVoter 가 계층적 Role 을 지원하도록 바꾸기


`AbstractServiceVoter` 클래스에는 인증된 사용자 정보를 이용해서 `Permission` `Set` 을 반환하는 `extractPermissions` 메소드가 있다. 이 메소드를 바꿔서 계층 `Role`을 지원하도록 하겠다.

### AbstractServiceVoter 에 Role tree 탐색 기능 구현하기

`AbstractServiceVoter` 의 `extractPermissions` 메소드를 아래처럼 바꾼다.

```java
protected Set<Permission> extractPermissions(Authentication authentication, RequestedServiceMeta requestedServiceMeta) {
    Set<Permission> permissions = new HashSet<>();
    Set<Role> roles = new HashSet<>();

    for (GrantedAuthority authority : authentication.getAuthorities()) {
        roles.add(roleProvider.getRole(authority.getAuthority()));
    }

    roles = travelRoleHierarchy(roles);

    for (Role role : roles) {
        if (role == null)
            continue;

        Optional.ofNullable(role.getPermissions()).filter(perms -> perms.stream().anyMatch(
                perm -> perm.getSecuredObject().getSecuredObjectType() == SecuredObjectType.SERVICE
                        && perm.getSecuredObject().getSecuredObjectId().equals(requestedServiceMeta.getServiceName())
        )).ifPresent(permissions::addAll);
    }

    return permissions;
}
```
`Role`의 계층 트리를 탐색할 수 있도록 재귀호출로 탐색 메소드를 구현했다.

```java
private Set<Role> travelRoleHierarchy(Set<Role> roles) {
    if (roles.size() == 0) return roles;

    Set<Role> tempRoles = new HashSet<>();

    for (Role role : roles) {
        if (role == null || role.getParentRole() == null || roles.contains(role.getParentRole())) {
        } else {
            tempRoles.add(role.getParentRole());
        }
    }

    roles.addAll(travelRoleHierarchy(tempRoles));

    return roles;
}
```
### Role 도메인 객체의 hashCode와 equals 메소드 재정의

`Role`은 내부의 `roleId`가 같으면 같은 객체라고 봐야 한다. 그렇지 않으면 `Set<Role>`ß은 같은 id를 가진 Role 객체가 존재할 수 있다.

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
    @JoinColumn(name = "parent_role_id")
    private Role parentRole;

    @OneToMany
    private Set<Permission> permissions;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Role role = (Role) o;
        return Objects.equals(roleId, role.roleId);
    }

    @Override
    public int hashCode() {
        return Objects.hash(roleId);
    }
}
```

### DenialFirstServiceVoterHierarchyTest 다시 돌려보기

![](/assets/2018-09-07-spring-security-service-4/2018-09-07-spring-security-service-4_162233.png)

# 마무리
여기까지 사용자 인증부터 권한 체크 모듈까지 우리가 정의한 `Role` 검증 로직을 가지고 구현을 해봤다. 이 코드가 실제 비즈니스에 사용할 수 있으려면 더 보완이 필요하겠지만, 전체적인 개념을 잡고 스프링 시큐리티에 어느 부분을 확장해서 각 도메인에 맞는 보안 서비스를 구축할 수 있는지 감을 잡을 수 있는 가이드가 됐으면 좋겠다.

처음 시작할 때는 사용자와 권한 관리를 하는 UI까지 구현해보려고 했는데 시간 압박으로 하지는 못했다. 여기에 기록이 될지는 모르겠지만 개인적으로 만들어보기는 할 것인데 그 과정에서 알게 된 점들은 꾸준히 공유할 것이다.

----
# 연관된 포스트

[Spring Security로 Security 서비스 구축하기 3](/spring/security/2018/09/06/spring-security-service-3.html)

[Spring Security로 Security 서비스 구축하기 2](/spring/security/2018/08/15/spring-security-service-2.html)

[Spring Security로 Security 서비스 구축하기 1](/spring/security/2018/07/31/spring-security-service-1.html)


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