---
layout: post
title: Spring Security로 Security 서비스 구축하기 4
description: Spring Security voter 구현하기 스프링 시큐리티 스프링시큐리티 access
date:   2018-09-06 17:30:00 +0900
author: Jeongjin Kim
categories: Spring Security
tags:	spring security spring-security 스프링 스프링시큐리티 보안 시큐리티 reference 레퍼런스 따라하기
---

# 계층구조 Role

RoleRepository 에서 계층적인 자료를 잘 가져오는지 테스트를 먼저 한다.

RoleRepositoryTest를 만들자.

`RoleRepositoryTest.java`

```java
package cothe.security.core.repositories;

@RunWith(SpringRunner.class)
@SpringBootTest
public class RoleRepositoryTest {
    @Autowired
    RoleRepository roleRepository;

    @Test
    public void loadHierarchicalRole() {
        Role role1 = Role.builder().roleId("role1").build();
        Role role2 = Role.builder().roleId("role2").parentRole(role1).build();
        Role role3 = Role.builder().roleId("role3").parentRole(role2).build();

        roleRepository.save(role1);
        roleRepository.save(role2);
        roleRepository.save(role3);

        Optional<Role> role3_op = roleRepository.findById("role3");

        Role role = role3_op.orElse(null);

        assertEquals(Objects.requireNonNull(role).getParentRole().getParentRole().getRoleId(), "role1");
    }
}
```