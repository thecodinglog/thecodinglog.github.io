# Spring Data Auditing
### 빈 등록
```java
@configuration
@EnableJpaAuditing
public class JpaAuditingConfiguration{
    @Bean
    public AuditorAware<String> auditorProvider(){
        return () -> session().getUserId();
    }
}
```
### 사용
```java
@CreatedDate
@LastModifiedDate
@CreatedBy
@LastModifiedBy
```

```java
@Entity
public class Member(){
    @Id @GeneratedValue
    private Long id;
    private String name;

    @CreatedDate
    private LocalDateTime createdDate;
    ....
}
```

이렇게 하면 되긴 한데 중복

BaseEntity 를 쓰면 상속받아서 코드상 중복을 없앨 수 있다.

하이버네이트 Envers
- 하이버네이트 핵심 모듈
- JPA 스펙에 정의된 모든 매핑 감사
- 엔티티의 변경 이력을 자동으로 관리

