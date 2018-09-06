---
layout: post
title: Spring Security로 Security 서비스 구축하기 3
description: Spring Security voter 구현하기
date:   2018-09-06 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---
# ViewVoter 보완

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
### RequestedViewExtractor 구현

`ViewVoter`에서 얼렁뚱땅 넘어간것이 있는데 `String targetView = (String) object;` 이 구문이다.

vote 메소드에 어떤 타입이 오는지도 모르는데 묻지도 따지지도 않고 `String` 으로 강제 형변환을 했다. 실제 이 코드가 웹에서 동작하게 되면 99.99% 캐스팅 예외가 발생할 것이다.

`Object`는 접근하고자 하는 대상물이다. 이 대상물이 어느 형태로 올 것인지는(웹에서 동작하게 되면 org.springframework.security.web.FilterInvocation 이 될 확률이 높다) 누가 호출하냐에 따라 다르다. 하지만 이 메소드는 같은 동작을 하도록 보장을 해야 하는데 이때 glue 역할로 좋은 것이 `Interface`이다. 어떤 것이 필요한지만 정의하고 실제 구현은 이 vote를 사용하는 곳에서 주입하도록 처리하는 것이다.

```java
package cothe.security.access.vote;

public class ViewVoter implements AccessDecisionVoter<Object> {

    private RoleProvider roleProvider;
    private RequestedViewMetaExtractor requestedViewMetaExtractor;

    public ViewVoter(RoleProvider roleProvider, RequestedViewMetaExtractor requestedViewMetaExtractor) {
        this.roleProvider = roleProvider;
        this.requestedViewMetaExtractor = requestedViewMetaExtractor;
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
        Assert.notNull(this.requestedViewMetaExtractor, "There is no RequestedViewExtractor.");

        if (!authentication.isAuthenticated()) {
            return ACCESS_DENIED;
        }

        String targetView = Optional.ofNullable(this.requestedViewMetaExtractor.extractViewMeta(object))
                .map(RequestedViewMeta::getViewName).orElse(null);
        if (targetView == null) {
            return ACCESS_ABSTAIN;
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

이렇게 변경하면 당연히 테스트가 실패할 것이다. 테스트가 성공하도록 `MockRequestedViewMetaExtractor` 를 구현하여 변경하자.

`MockRequestedViewMetaExtractor.java`

```java
package cothe.security.mock;

public class MockRequestedViewMetaExtractor implements RequestedViewMetaExtractor {
    @Override
    public RequestedViewMeta extractViewMeta(Object object) {
        return new RequestedViewMeta((String)object);
    }
}

```
`ViewVoterTest.java`

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
        viewVoter = new ViewVoter(new MockRoleProvider(permissions), new MockRequestedViewMetaExtractor());
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

    @Test
    public void permission이null이면(){
        ViewVoter nullPermissionViewVoter = new ViewVoter(new MockRoleProvider(null), new MockRequestedViewMetaExtractor());

        String targetView = "default_object";
        int voteResult = nullPermissionViewVoter.vote(authentication, targetView, null);
        assertTrue(voteResult < 0);
    }
}
```
`AccessDecisionManagerTest.java`
```java
package cothe.security.access.vote;

@RunWith(SpringRunner.class)
@WithMockSecuredUser(username = "admin", name = "admin", roles = "default_view_permission")
public class AccessDecisionManagerTest {
    private Set<Permission> permissions;
    private Authentication authentication;
    private ViewVoter viewVoter;
    private AccessDecisionManager accessDecisionManager;

    @Before
    public void setUp() {
        permissions = Stream.of(
                new Permission("default_view_permission", "default_view_permission", null,
                        new SecuredObject("default_object", "default_object", SecuredObjectType.VIEW))
        ).collect(Collectors.toSet());
        viewVoter = new ViewVoter(new MockRoleProvider(permissions), new MockRequestedViewMetaExtractor());
        authentication = SecurityContextHolder.getContext().getAuthentication();
        accessDecisionManager = new AffirmativeBased(
                Arrays.asList(
                        viewVoter
                )
        );
    }

    @Test
    public void decideTest() {
        String targetView = "default_object";
        accessDecisionManager.decide(authentication, targetView, null);

    }
}

```

`ViewVoter.java`

```java
package cothe.security.access.vote;

public class ViewVoter implements AccessDecisionVoter<Object> {

    private RoleProvider roleProvider;
    private RequestedViewMetaExtractor requestedViewMetaExtractor;

    public ViewVoter(RoleProvider roleProvider, RequestedViewMetaExtractor requestedViewMetaExtractor) {
        this.roleProvider = roleProvider;
        this.requestedViewMetaExtractor = requestedViewMetaExtractor;
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
        Assert.notNull(this.requestedViewMetaExtractor, "There is no RequestedViewExtractor.");

        if (!authentication.isAuthenticated()) {
            return ACCESS_DENIED;
        }

        String targetView = Optional.ofNullable(this.requestedViewMetaExtractor.extractViewMeta(object))
                .map(RequestedViewMeta::getViewName).orElse(null);
        if (targetView == null) {
            return ACCESS_ABSTAIN;
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
# ServiceVoter 구현

요청 중 View 자원에 대한 접근 권한 체크는 ViewVoter에서 처리하도록 했다. 이제 그 이외 자원 요청들에 대해 권한 체크를 하는 Voter를 구현할 것이다. 하지만 그 전에 지금 구현하고자 하는 접근 제어 시스템의 구조와 동작 방법을 다시 정의하고자 한다.

![](/assets/2018-07-31-spring-security-service-2/security_domain.png)

먼저 앞선 포스트에서 정의한 도메인 클래드들의 관계를 대략 표현한 다이어그램이다. User는 Role 오브젝트들을 가지고 있고 이것은 Spring Security 의 Authorties 와 연관된다.

Role 은 Permission 을 가지고 있는데 다시 말하면 특정 역할이 가지는 권한들을 정의한다고 보면 된다.

이 Permission은 보호된 오브젝트(SecuredObject)와 1:1 매핑된다. 이는 같은 오브젝트라도 오브젝트에 대한 권한을 정의하는 것은 다양하다는 뜻이다. 예를 들어 _사용자 관리_ 라는 서비스가 있을 때 같은 _서비스 오브젝트_ 에 대해서 _사용자를 추가 할 수 있는 권한_ 이 있을 수 있고, _조회만 가능한 권한_ 이 있을 수 있다.

Permission에서 눈여겨 봐야 할 것은 String 타입의 **permission** 필드이다.

이 필드는 요청한 오브젝트에 대한 권한을 정의하는 JSON 형태의 문자열 데이터를 가지고 있다. JSON 데이터의 구조를 정의해 보자.

```json
{
  "permissionType": "allow",      // ["allow", "deny"]
  "definitions": [                // operation과 params의 묶음을 defition 이라고 명명했다.
    {
      "operation": "order,save",  // 매핑할 method 명 리스트라고 보면 된다.
      "params": [                 // method와 함께 비교해볼 파라미터들을 정의한다.
        {
          "genre": "dance",       // { } 사이에 나열된 정보는 and 조건이다. 즉 둘다 맞아야 매치된다.
          "since": "1998"
        },                        // [ ] 사이에 나열된 정보는 or 조건이다. 즉 둘중 하나만 매치되도 결과가 매치됨이다.
        {
          "genre": "/j.*/",       // 문자열 양 끝에 / 를 넣으면 정규식으로 인식한다.
          "since": "1920"
        }
      ]
    },
    {
      "operation": "search",
      "params": [
        {
          "genre": "dance"
        }
      ]
    }
  ]
}
```
이 정의 구조에서 약속된 이름은 _"permissionType", "definitions", "operation", "params"_ 이다.

위 설정을 기준으로 요청에 대한 접근 권한 체크과정을 시뮬레이션 해보자.

보호된 오브젝트가 음반을 조회하고 주문하는 서비스(_MusicAlbumOrderService_) 라고 가정한다. 사용자는 _MusicAlbumOrderService_ 에 어떤 음반들이 있는지 조회를 한다. 조회를 할 때 조회조건에 장르를 'classic' 이라고 넣을 경우 serviceId는 _MusicAlbumOrderService_, operation은 'search' 그에 따른 파라미터는 'genre' 에 'classic' 인 요청이 오게된다. 이 요청은 ServiceVoter가 접근 허용 여부를 결정한다. 결정과정을 나열해보면
1. 요청자가 가진 역할이 요청한 MusicAlbumOrderService 를 가지고 있는지 확인한다.
2. 있으면 그 오브젝트의 Permission 을 가져온다.
3. 요청한 operation search 를 만족하는 definition(operation과 params 를 definition 이라 명명했다) 을 가져온다.
4. definition에 정의된 파라미터 목록과 요청한 파라미터가 매칭되는지 확인한다. 이번 요청은 만족하는 값이 없다.
5. 매칭되는 것이 있으면 그 것을 허용할 것인지, 거부할 것인지 permissionType에 의거 결정한다.

이런 절차로 vote 메소드를 수행시킨다. 이 절차대로 Voter를 구현해보자. 

`AbstractServiceVoter.java`

```java
package cothe.security.access.vote;

@Slf4j
public abstract class AbstractServiceVoter implements AccessDecisionVoter<Object> {
    private final RoleProvider roleProvider;
    private final RequestedServiceMetaExtractor requestedServiceMetaExtractor;
    private static final Gson gson = new Gson();

    /**
     * AbstractServiceVoter 객체를 생성하기 위해서는 두 오브젝트가 필요합니다. RoleProvider 는 사용자의 권한 도메인 오브젝트를 반환합니다.
     * RequestedServiceMetaExtractor 는 요청한 오브젝트를 기반으로 권한 체크를 할 수 있는 표준화된 클래스(RequestedServiceMeta)로 반환합니다.
     *
     * @param roleProvider roleId를 입력받아 Role 도메인 객체를 생성하여 리턴하는 인터페이스
     * @param requestedServiceMetaExtractor 클라이언트가 요청한 정보를 RequestedServiceMeta 오브젝트를 생성하여 반환하는 인터페이스
     */
    @SuppressWarnings("WeakerAccess")
    protected AbstractServiceVoter(RoleProvider roleProvider, RequestedServiceMetaExtractor requestedServiceMetaExtractor) {
        this.roleProvider = roleProvider;
        this.requestedServiceMetaExtractor = requestedServiceMetaExtractor;
    }

    /**
     * 스프링 시큐리티에서 주는 권한 설정 정보는 무시합니다.
     */
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
        Assert.notNull(this.requestedServiceMetaExtractor, "There is no RequestedServiceMetaExtractor.");

        if (!authentication.isAuthenticated()) {
            return ACCESS_DENIED;
        }

        RequestedServiceMeta requestedServiceMeta = this.requestedServiceMetaExtractor.extractRequestedServiceMeta(object);

        return doVote(authentication, requestedServiceMeta);

    }

    /**
     * permission 의 type 에 따라 어떻게 결과를 낼 것인지 구체적인 전략을 추상 메소드를 구현해서 사용할 수 있습니다.
     */
    abstract int doVote(Authentication authentication, RequestedServiceMeta requestedServiceMeta);

    /**
     * Permission n 개를 파라미터로 넘기면 n 개의 PermissionDescription 으로 반환합니다.
     * parsing 할 permission json string 이 null 이거나 형식에 맞지 않으면 null 을 반환 리스트에 추가합니다.
     */
    @SuppressWarnings("WeakerAccess")
    protected List<PermissionDescription> parsePermissionDescriptionScripts(Set<Permission> permissions) {
        List<PermissionDescription> permissionDescriptions = new ArrayList<>();
        PermissionDescription permissionDescription;
        for (Permission permission : permissions) {
            permissionDescription = null;
            try {
                permissionDescription = gson.fromJson(permission.getPermission(), PermissionDescription.class);
            } catch (JsonSyntaxException e) {
                log.error("권한정보를 읽는 중 오류가 발생했습니다. Permission[{}]가 JSON 형식인지 확인하세요.", permission.getPermissionId());
            }
            permissionDescriptions.add(permissionDescription);
        }

        return permissionDescriptions;
    }

    /**
     * Permission 에 정의된 Definition 들을 순회하면서 사용자 요청과 매치된 것을 찾으면 즉시 true 를 반환합니다.
     */
    @SuppressWarnings("WeakerAccess")
    protected boolean doesPermissionDescriptionMatchRequestedService(
            @NonNull PermissionDescription permissionDescription
            , @NonNull RequestedServiceMeta requestedServiceMeta) {
        List<Definition> definitions = permissionDescription.getDefinitions();
        for (Definition definition : definitions) {
            // 사용자가 요청한 Operation 을 definition 이 정의하고 있는지 확인
            if (doesDefinitionHaveRequestedOperation(definition, requestedServiceMeta)) {
                // Definition 에 정의된 파라미터와 요청된 파라미터가 매치되는지 확인,
                if (doesDefinitionMatchRequestedParams(definition, requestedServiceMeta)) {
                    return true;
                }
            }
        }
        return false;
    }

    /**
     * Definition 에 정의된 Operation 이 요청한 Operation 를 포함하면 true 를 반환합니다.
     */
    private boolean doesDefinitionHaveRequestedOperation(Definition op, RequestedServiceMeta requestedServiceMeta) {
        return permissionExpressionMatcher(op.getOperation(), requestedServiceMeta.getOperation());
    }

    /**
     * Definition 에 정의된 파라미터를 순회하면서 사용자 요청과 매치되면 true 를 반환합니다.
     */
    private boolean doesDefinitionMatchRequestedParams(Definition definition, RequestedServiceMeta requestedServiceMeta) {
        for (Map<String, String> param : definition.getParams()) {
            if (doesOperationParamMatchRequestedParam(param, requestedServiceMeta.getParams())) {
                return true;
            }
        }
        return false;
    }

    /**
     * 파라미터의 엔트리를 순회하면서 사용자 요청과 매치되면 true 를 반환합니다.
     * Permission 에는 정의가 된 파라미터 엔트리가 요청에 존재하지 않으면 즉시 false 를 반환합니다.
     */
    private boolean doesOperationParamMatchRequestedParam(Map<String, String> permissionParam, Map<String, String> requestedParam) {
        for (Map.Entry<String, String> entry : permissionParam.entrySet()) {
            String requestedValue = requestedParam.get(entry.getKey());
            if (requestedValue == null) {
                return false;
            }

            if (!doesParamMatchRequestedParam(entry.getValue(), requestedValue))
                return false;
        }
        return true;
    }

    private boolean doesParamMatchRequestedParam(String permissionParamEntry, String requestedValue) {
        return permissionExpressionMatcher(permissionParamEntry, requestedValue);
    }

    //todo: parent role 에서 permission 가져오는 정책은?
    /**
     * 사용자는 여러 Role 을 가질 수 있기 때문에 Role 이 가지고 있는 퍼미션이 중복될 수 있다.
     * extractPermissions 는 롤들이 가지고 있는 퍼미션을 모두 가져와서 중복되지 않는 Set 타입으로 반환한다.
     */
    @SuppressWarnings("WeakerAccess")
    protected Set<Permission> extractPermissions(Authentication authentication, RequestedServiceMeta requestedServiceMeta) {

        Set<Permission> permissions = new HashSet<>();

        for (GrantedAuthority authority : authentication.getAuthorities()) {
            Role role = roleProvider.getRole(authority.getAuthority());
            if (role == null)
                continue;

            Optional.ofNullable(role.getPermissions()).filter(perms -> perms.stream().anyMatch(
                    perm -> perm.getSecuredObject().getSecuredObjectType() == SecuredObjectType.SERVICE
                            && perm.getSecuredObject().getSecuredObjectId().equals(requestedServiceMeta.getServiceName())
            )).ifPresent(permissions::addAll);
        }
        return permissions;
    }

    /**
     * permission String 은 <kbd>,</kbd> 로 구분된 리스트일 수 있습니다.
     * 따라서 <kbd>,</kbd> 로 개별 item 으로 분리하여 순회하고 매치되면 즉시 true 를 반환합니다.
     *
     * 개별 표현식 item 이 <kbd>/</kbd> 로 시작하고 <kbd>/</kbd> 로 끝나면 내부 스트링을 정규표현식으로 인식합니다.
     */
    private boolean permissionExpressionMatcher(String permissionExpression, String targetValue) {
        String[] permissionParamEntryValues = permissionExpression.split(",");
        for (String permissionParamEntryValue : permissionParamEntryValues) {
            if (permissionParamEntryValue.startsWith("/") && permissionParamEntryValue.endsWith("/")) {
                String rex = permissionParamEntryValue.substring(1, permissionParamEntryValue.length() - 1);
                if (targetValue.matches(rex)) {
                    return true;
                }

            } else {
                if (permissionParamEntryValue.equals(targetValue)) {
                    return true;
                }
            }
        }
        return false;
    }
}

```

`ServiceVoter` 의 구현체를 작성한다. 사용자가 가지고 있는 `Permission` 중 `cothe.security.access.PermissionType#PERMISSION_TYPE_DENY` 타입에 매치되는 것이 있으면 즉시 `org.springframework.security.access.AccessDecisionVoter#ACCESS_DENIED` 를 반환하도록 구현했다.

`DenialFirstServiceVoter.java`

```java
package cothe.security.access.vote;

@Slf4j
public class DenialFirstServiceVoter extends AbstractServiceVoter {
    public DenialFirstServiceVoter(RoleProvider roleProvider, RequestedServiceMetaExtractor requestedServiceMetaExtractor) {
        super(roleProvider, requestedServiceMetaExtractor);
    }

    @Override
    int doVote(Authentication authentication, RequestedServiceMeta requestedServiceMeta) {
        List<PermissionDescription> permissionDescriptions = parsePermissionDescriptionScripts(extractPermissions(authentication, requestedServiceMeta))
                .stream().filter(Objects::nonNull).sorted((o1, o2) -> {
                    if (o1.getPermissionType().equals(PERMISSION_TYPE_DENY) && o2.getPermissionType().equals(PERMISSION_TYPE_ALLOW)) {
                        return -1;
                    } else if (o1.getPermissionType().equals(PERMISSION_TYPE_ALLOW) && o2.getPermissionType().equals(PERMISSION_TYPE_DENY)) {
                        return 1;
                    } else {
                        return 0;
                    }
                }).collect(Collectors.toList());

        for (PermissionDescription permissionDescription : permissionDescriptions) {
            if (doesPermissionDescriptionMatchRequestedService(permissionDescription, requestedServiceMeta)) {
                if (permissionDescription.getPermissionType().equals(PERMISSION_TYPE_DENY)) {
                    log.debug("{}-{}는 거부되었습니다.",requestedServiceMeta.getServiceName(), requestedServiceMeta.getOperation());

                    return ACCESS_DENIED;
                } else if (permissionDescription.getPermissionType().equals(PERMISSION_TYPE_ALLOW)) {
                    return ACCESS_GRANTED;
                }
            }
        }
        return ACCESS_DENIED;
    }
}
```
permission String 를 파싱하면 리턴되는 도메인 클래스들을 구현한다.

`Definition.java`

```java
package cothe.security.access;

@Getter
public class Definition {
    /**
     * <kbd>,</kbd>로 구분된 Operation 이 올 수 있다.
     */
    private final String operation;
    private final List<Map<String, String>> params;

    public Definition(String operation, List<Map<String, String>> params) {
        this.operation = operation;
        this.params = params;
    }
}
```
`PermissionDescription.java`
```java
package cothe.security.access;

@Getter
public class PermissionDescription {
    private final String permissionType;
    private final List<Definition> definitions;

    public PermissionDescription(String permissionType, List<Definition> definitions) {
        this.permissionType = permissionType;
        this.definitions = definitions;
    }
}
```
`PermissionType.java`
```java
package cothe.security.access;

public interface PermissionType {
    String PERMISSION_TYPE_ALLOW = "allow";
    String PERMISSION_TYPE_DENY = "deny";
}
```

스프링 시큐리티에서 넘겨주는 요청 오브젝트를 우리가 정의한 요청 클래스로 변환하는 인터페이스를 추가한다.
이것의 구현은 클라이언트가 어떤 시스템인가에 따라서 적절히 구현한다.

`RequestedServiceMetaExtractor.java`

```java
package cothe.security.access;

public interface RequestedServiceMetaExtractor {
    RequestedServiceMeta extractRequestedServiceMeta(Object object);
}
```
우리가 정의한 시큐리티 서비스의 요청 메타정보 클래스를 정의한다.

`RequestedServiceMeta.java`
```java
package cothe.security.access;

@Getter
@AllArgsConstructor
@NoArgsConstructor
public class RequestedServiceMeta {
    private String serviceName;
    private String operation;
    private Map<String, String> params;
}
```

# 테스트

테스트를 하기위해 몇 가지 Mock Class를 만들어서 사용할 것이다. 
### RoleProvider 
role id 와 Permission 리스트를 받아서 저장하고 있다가 요청시에 반환하는 역할을 한다.

`MockRepositoryRoleProvider.java`
```java
package cothe.security.mock;

public class MockRepositoryRoleProvider implements RoleProvider {
    private Map<String, Role> roles = new HashMap<>();

    @Override
    public Role getRole(String roleId) {
        return roles.get(roleId);
    }

    public void putRole(String roleId, Set<Permission> permissions){
        roles.put(roleId, new Role(roleId, roleId, null, permissions));
    }
}
```

### RequestedServiceMetaExtractor

`extractRequestedServiceMeta` 메소드의 파라미터 타입이 `RequestedServiceMeta` 이면 그대로 반환하고 아니면 디폴트 값을 임의로 만들어서 반환한다.

`MockRequestedServiceMetaExtractor.java`

```java
package cothe.security.mock;

public class MockRequestedServiceMetaExtractor implements RequestedServiceMetaExtractor {
    @Override
    public RequestedServiceMeta extractRequestedServiceMeta(Object object) {
        if(object instanceof RequestedServiceMeta){
            return (RequestedServiceMeta) object;
        }else {
            Map<String, String> param = new HashMap<>();
            param.put("device","mobile");
            RequestedServiceMeta requestedServiceMeta = new RequestedServiceMeta(
                    "service1",
                    "save",
                    param
            );
            return requestedServiceMeta;
        }
    }
}
```

### ServiceVoterTest

테스트를 하기 위해 준비해야 할 데이터가 상당히 많은 편이다. DB나 파일로 된 것을 직접 읽어서 사용하지 않고 도메인 오브젝트를 직접 만들어서 테스트에 사용한다.

```java
package cothe.security.access.vote;

@RunWith(SpringRunner.class)
@WithMockSecuredUser(username = "admin", name = "admin", roles = "role1,role2,role3")
public class DenialFirstServiceVoterTest {
    private Authentication authentication;
    private DenialFirstServiceVoter denialFirstServiceVoter;
    private MockRepositoryRoleProvider mockRepositoryRoleProvider = new MockRepositoryRoleProvider();
    private static Gson gson = new Gson();

    @Before
    public void setUp() {
        mockRepositoryRoleProvider.putRole("role1",
                Stream.of(
                        new Permission("permission1", "permission1", serializedPermissionDescriptionOf("permission1"),
                                new SecuredObject("object1", "object1", SecuredObjectType.SERVICE)),
                        new Permission("permission2", "permission2", serializedPermissionDescriptionOf("permission2"),
                                new SecuredObject("object2", "object2", SecuredObjectType.SERVICE))
                ).collect(Collectors.toSet()));

        mockRepositoryRoleProvider.putRole("role2",
                Stream.of(
                        new Permission("permission2-1", "permission2-1", serializedPermissionDescriptionOf("permission2-1"),
                                new SecuredObject("object2", "object2", SecuredObjectType.SERVICE)),
                        new Permission("permission3", "permission3", serializedPermissionDescriptionOf("permission3"),
                                new SecuredObject("object3", "object3", SecuredObjectType.SERVICE))
                ).collect(Collectors.toSet()));

        authentication = SecurityContextHolder.getContext().getAuthentication();

        denialFirstServiceVoter = new DenialFirstServiceVoter(
                mockRepositoryRoleProvider,
                new MockRequestedServiceMetaExtractor()
        );
    }
    
    private String serializedPermissionDescriptionOf(String permissionId) {
        PermissionDescription permissionDescription = null;
        switch (permissionId) {
            case "permission1":
                permissionDescription = new PermissionDescription(
                        PERMISSION_TYPE_ALLOW,
                        Arrays.asList(
                                new Definition("save",
                                        Arrays.asList(new HashMap<String, String>() {
                                            {
                                                put("device", "mobile");
                                            }
                                        }))
                                , new Definition("/input.*/,persist",
                                        Arrays.asList(
                                                new HashMap<String, String>() {
                                                    {
                                                        put("device", "pc");
                                                    }
                                                }
                                                , new HashMap<String, String>() {
                                                    {
                                                        put("device", "pda");
                                                    }
                                                }
                                        )
                                )
                        )
                );
                break;
            case "permission2":
                permissionDescription = new PermissionDescription(
                        PERMISSION_TYPE_ALLOW,
                        Arrays.asList(
                                new Definition("search",
                                        Arrays.asList(new HashMap<String, String>() {
                                            {
                                                put("os", "mac");
                                                put("version", "6");
                                            }
                                        }))
                                , new Definition("click",
                                        Arrays.asList(
                                                new HashMap<String, String>() {
                                                    {
                                                        put("device", "pc");
                                                    }
                                                }
                                        )
                                )
                        )
                );
                break;
            case "permission2-1":
                permissionDescription = new PermissionDescription(
                        PERMISSION_TYPE_DENY,
                        Arrays.asList(
                                new Definition("click",
                                        Arrays.asList(
                                                new HashMap<String, String>() {
                                                    {
                                                        put("device", "pc");
                                                    }
                                                }
                                        )
                                )
                        )
                );
                break;
        }
        return gson.toJson(permissionDescription);
    }

    @Test
    public void successVoting() {
        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object1",
                        "save",
                        new HashMap<String, String>() {
                            {
                                put("device", "mobile");
                            }
                        }
                ), null) > 0);

        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object1",
                        "save",
                        new HashMap<String, String>() {
                            {
                                put("device", "mobile");
                                put("os", "mac");
                            }
                        }
                ), null) > 0);

        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object1",
                        "input1",
                        new HashMap<String, String>() {
                            {
                                put("device", "pc");
                                put("os", "mac");
                            }
                        }
                ), null) > 0);

        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object1",
                        "input2",
                        new HashMap<String, String>() {
                            {
                                put("device", "pda");
                                put("os", "mac");
                            }
                        }
                ), null) > 0);

        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object1",
                        "persist",
                        new HashMap<String, String>() {
                            {
                                put("device", "pda");
                                put("os", "mac");
                            }
                        }
                ), null) > 0);

        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object2",
                        "search",
                        new HashMap<String, String>() {
                            {
                                put("device", "pda");
                                put("os", "mac");
                                put("version", "6");

                            }
                        }
                ), null) > 0);
    }

    @Test
    public void denyVoting() {
        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object1",
                        "save",
                        new HashMap<String, String>() {
                            {
                                put("device", "pc");
                            }
                        }
                ), null) < 0);

        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object1",
                        "input",
                        new HashMap<String, String>() {
                            {
                                put("device", "mobile");
                                put("os", "mac");
                            }
                        }
                ), null) < 0);

        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object2",
                        "input",
                        new HashMap<String, String>() {
                            {
                                put("device", "mobile");
                                put("os", "mac");
                            }
                        }
                ), null) < 0);
        assertTrue(denialFirstServiceVoter.vote(authentication,
                new RequestedServiceMeta(
                        "object2",
                        "click",
                        new HashMap<String, String>() {
                            {
                                put("device", "pc");
                            }
                        }
                ), null) < 0);
    }
}
```

# 마무리
권한 관리는 어느 수준으로 관리하느냐에 따라 시스템 복잡도가 매우 높아진다. 엔터프라이즈 애플리케이션에는 명령 또는 동작에 대한 권한 관리가 필수이다. 이런 관리 기능을 프레임워크 레벨에서 지원해주지 않으면 개발 공수가 현저히 높아지고 권한의 효율적인 관리가 힘들어진다. 스프링 시큐리티의 애노테이션을 이용해서 메서드 보안을 실현해도 상관없지만, 권한의 복잡도가 어느 정도 높아지면 관리하기가 매우 힘들 것이라고 생각한다.

지금까지 `ViewVoter`와 `ServiceVoter`를 대략 완성했다. 부족한 부분은 Role 이 계층적인 구조로 되어 있지만, 특별히 그 부분에 대해서 구현하지 않았다. 부모의 Role과 자식 Role 이 상충할 때 어떻게 처리할지 정책을 정하고 이에 맞게 구현해야 한다. 특정 조건에 따라서 정책을 다르게 써야 할 수도 있다. 이런 조건을 만족 할 수 있도록 다음 포스트부터 진행하겠다.


