---
layout: post
title: Enum으로 JPA 상속관계 구분 커럼 지정 
description: Enum으로 JPA 상속관계 구분 커럼 지정
date:   2019-09-09 13:00:00 +0900
author: Jeongjin Kim
categories: JPA
tags:	JPA DiscriminatorColumn
---

클래스의 상속관계를 JPA로 표현하기위해서 `@DiscriminatorColumn` 애노테이션으로 부모 클래스의 구분 컬럼을 지정하고 `@DiscriminatorValue` 애노테이션으로 자식 클래스가 사용할 구분자 값을 지정한다.

`@DiscriminatorValue`는 `String` 타입을 Value 로 지정하게 되어있다.

일반적으로 구분자 또는 타입 같은 값은, 값 리스트들을 체계적으로 관리하기위해 `enum` 으로 선언해 사용하고는 하는데 이것을 JPA에 바로 적용할 수가 없다.

이 포스트에서는 `enum`을 사용하면서 JPA의 `@DiscriminatorValue` 에 사용할 수 있는 우회적인 방법을 알아본다.

먼저 부모 클래스에 `@DiscriminatorColumn` 애노테이션으로 구분자로 사용할 컬럼명과 `discriminatorType` 을 지정한다.

```java
@Entity
@DiscriminatorColumn(name = "GroupType",
        discriminatorType = DiscriminatorType.STRING)
public abstract class JobGroup {
    @Id
    private String id;
}
```

`enum`에 필요한 값들을 선언한다. `enum` 생성자에 이름과 값이 같도록 강제하여 혼선이 오지 않도록 예방코드를 추가하였다.

```java
public enum GroupType {
    CONCURRENT(Values.CONCURRENT),
    SEQUENTIAL(Values.SEQUENTIAL),
    EXCLUSIVE(Values.EXCLUSIVE);

    private String value;

    GroupType(String val) {
        if (!this.name().equals(val))
            throw new IllegalArgumentException("Incorrect use of GroupType");
    }

    public static class Values {
        public static final String CONCURRENT = "CONCURRENT";
        public static final String SEQUENTIAL = "SEQUENTIAL";
        public static final String EXCLUSIVE = "EXCLUSIVE";
    }
}
```

자식 클래스에 `@DiscriminatorValue` 애노테이션으로 구분자 값을 지정한다.
```java
@Entity
@DiscriminatorValue(value = GroupType.Values.EXCLUSIVE)
public class ExclusiveJobGroup extends JobGroup {
}
```

## 참고
> [stakoverflow](https://stackoverflow.com/questions/3639225/single-table-inheritance-strategy-using-enums-as-discriminator-value)