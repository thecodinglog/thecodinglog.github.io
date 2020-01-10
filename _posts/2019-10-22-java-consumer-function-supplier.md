---
layout: post
title: java Consumer 와 Function, Supplier interface
description: Consumer 와 Function, Supplier interface
date:   2019-10-22 13:00:00 +0900
author: Jeongjin Kim
categories: Java
tags:	Java Consumer Function Supplier
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

# Consumer

**파라미터 한 개를 입력받고 반환 값이 없는 인터페이스**이다. 다른 함수형 인터페이스와 다르게 **사이드 이펙트**를 이용해서
일을 처리하게 되어있는 것이 특징이다. 이런 형태, 즉 입력값을 받고 사이드 이펙트를 이용해서 일을 처리하는 인터페이스가 다수 있는데
`BiConsumer`, `IntConsumer` 등 입력 파라미터가 몇 개인지, 입력할 타입이 무엇인지에 따라서 추가로 정의되어 있다.

아래는 `Cunsumer` Interface 의 정의이다. 메소드 2개를 가지고 있는데 `accept` 는 입력된 파라미터로 실제 일을 하는 메소드이고
`andThen` 메소드는 `accept` 메소드를 수행하고 나서 실행할 명령을 입력 받아서 순차적으로 실행한다.

 ```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

`Consumer`를 사용하고 있는 대표적인 메소드가 `Iterable` Interface 의 `forEach` 메소드이다.
`forEach` 메소드는 컬렉션을 순회하면서 `aceept` 메소드를 실행한다. 

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

# Function
**파라미터 한 개를 받고 그 결과를 반환하는 인터페이스**이다. `Consumer` 인터페이스와 마찬가지로 `BiFunction`, `DoubleFunction` 등 
입력 파라미터의 타입과 개수, 리턴 타입에 따라서 유사한 인터페이스가 있다. 

아래는 `Function` Inteface 의 정의이다. `apply` 메소드는 입력된 파라미터로 일을 처리한 뒤 결과를 리턴하는 메소드이다.
`compose` 와 `andThen` 메소드는 원래 정의된 일을 하기 전(`compose`) 후(`andThen`) 에 추가로 해야 할 일을 입력받을 수 있는 메소드이다.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

# Supplier

입력값은 없고 **특정 타입을 리턴하는 기능만을 가진 인터페이스**이다. 

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

이 인터페이스를 사용하는 대표적인 메소드가 `Optional` Class 에 `orElseGet` 메소드이다.

```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```

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