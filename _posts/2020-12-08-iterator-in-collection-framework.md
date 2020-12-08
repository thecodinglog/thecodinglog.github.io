---
layout: post
title: Iterator 사용법과 Collection Framework
description: Collection Framework에서 제공하는 Iterator 어떻게 사용하는가?
date:   2020-12-08 08:00:00 +0900
author: Jeongjin Kim
categories: Java
tags:	Java Iterator Iterable Collection-Framework
---

# Collection Framework

**컬렉션 프레임워크**는 **데이터 그룹**을 **표준화된 방법**으로 **탐색**하고 **조작**하기 위한 통합 아키텍처입니다. 이는 컬렉션의 상세 구현과는 별개로 조작할 수 있도록 지원하기 때문에 일관된 방식으로 요소들을 다룰 수 있도록 많은 기능을 제공하고 있습니다. 이 글에서는 컬렉션 프레임워크의 핵심 인터페이스와 주요 기능, 유의할 점들을 예시를 통해서 안내하고, 컬렉션 순회를 위한 Iterator와 컬렉션 유틸리티를 소개합니다.

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

# 핵심 인터페이스들

컬렉션 프레임워크에는 엘리먼트 중복 저장 여부, 순서 저장 여부 등으로 구분하여 주요 인터페이스를 제공합니다.

![](/assets/2020-12-08-iterator-in-collection-framework/Untitled.png)


[리스트 1] 컬렉션 프레임워크의 주요 인터페이스들

`List`와 `Set`에서 공통으로 제공할 수 있는 기능은 `Collection` 인터페이스로 묶어 놨습니다. 사실상 컬렉션 프레임워크의 최상위 인터페이스라고 할 수 있습니다.

# List

**중복 저장을 허용**하고 **요소가 추가된 순서를 유지**합니다. 순서를 유지하고 있기 때문에 인덱스를 알면 바로 접근이 가능합니다. 

또한 `ListIterator` 인터페이스를 제공하는데 `Iterator` 인터페이스와 다르게 뒤 요소의 인덱스를 가져올 수 있어 리스트를 다룰 때 매우 유용하게 쓸 수 있습니다.

## 대표 클래스

### ArrayList

**배열을 기반**으로 만들어진 `List`입니다. 요소가 추가된 순서대로 배열에 차곡차곡 넣습니다. **중복 요소를 허용**하고 **순서를 보장**합니다. 

배열의 단점은 **한번 생성하면 크기를 변경할 수 없는 것**입니다. `ArrayList`는 사용자가 `List`의 크기를 신경 쓰지 않고 사용할 수 있지만, 내부적으로는 배열의 단점을 그대로 가지고 있습니다.  최초 `List`를 생성하면 요소 10개를 담을 수 있는 배열을 만들어 놓고 다 차면 자동으로 더 큰 배열을 생성합니다. 그리고 기존에 있던 값을 복사합니다.

즉, `List` 넣을 **요소 개수를 예상할 수 있으면 처음부터 알맞은 배열을 만들어 넣고 사용**하는 게 성능 면에서는 유리합니다. 하지만 메모리 공간은 비효율적으로 사용하겠지요.

```java
List list = new ArrayList(initialArraySize);
```

또한 `List` 중간에 새로운 값을 끼워 넣으려면 해당 인덱스 뒤에 위치하는 데이터를 모두 복사해야 하기 때문에 오버해드가 생기게 됩니다.

그래서 `ArrayList`는 **데이터를 차례대로 추가하고 리스트 중간에 데이터 삽입이 드물게 일어나는 경우에 사용**하는 것이 좋습니다.

### LinkedList

**Linked List 자료구조를 구현한 클래스**입니다. 요소를 추가하면 새로운 **노드**를 만들어서 추가합니다. 큰 공간을 차지하는 배열을 사용하지 않기 때문에 **메모리를 효율적으로 사용**할 수 있습니다.

Linked List이기 때문에 중간에 요소를 추가하여도 별다른 오버헤드 없이 수행할 수 있습니다. 

다만 이 `List`의 단점은 **특정 인덱스에 해당하는 요소에 접근하기 위해서는 시작 노드부터 차례대로 찾아서 가야** 하기 때문에 약간의 비용이 발생합니다. 또한 새 요소를 추가할 때 새로운 **메모리 공간을 할당**받아야 하므로 배열보다 비용이 더 발생합니다.

자료구조의 Linked List는 단방향 순회가 가능하게 되어있습니다. 컬렉션 프레임워크의 `LinkedList`는 Double Linked List로 **양방향**으로 순회가 가능하도록 구현되어 있습니다.

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

따라서 Linked List는 **요소 중간에 삽입, 삭제 등이 자주 일어나는 상황에서 사용**하는 것이 좋습니다.

# Set

**중복 저장을 허용하지 않는 컬렉션**입니다. 이름에서 알 수 있듯이 수학의 집합 개념을 구현하고 있는 인터페이스입니다.

## 대표 클래스

### HashSet

**HashMap을 기반**으로 만들어진 Set입니다. 수학의 집합처럼 같은 값은 중복해서 저장되지 않습니다. 왜냐하면, **값 자체를 HashMap의 Key로 사용**하기 때문입니다. 만약 add를 했는데 같은 값이 있다면 false를 반환합니다. 또한, `HashMap` 처럼 컬렉션에 추가된 순서대로 순회할 수 없습니다.

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

`add()`, `remove()`, `contains()`, `size()` 메소드는 **상수 시간에 처리**할 수 있는 성능을 내기 때문에 대량의 데이터를 다룰 때도 좋은 성능을 냅니다. 하지만 컬렉션을 순회하는 시간은 **해당 Set의 현재 크기에 더해서 해당 셋이 가지고 있는 용량과 비례**하기 때문에 초기에 Set을 만들 때 너무 큰 값을 넣으면 성능이 떨어지므로 유의해야 합니다.

> 요소 10,000개가 있는 Set을 순회하는데 초깃값을 15,000으로 했을 때는 1ms가 소요됐고 100,000,000으로 했을 때는 200ms가 소요됐습니다.

### TreeSet

**TreeMap을 기반**으로 만들어진 Set입니다. **유일한 값을 저장** 할 수 있을 뿐만 아니라, Set 생성 시 제공된 `Comparator`나 요소가 가지고 있는 `Comparator`를 이용해서 **정렬된 상태로 저장**할 수 있습니다. 

> 저장한 순서가 아니라 값이 **정렬된 순서**입니다.

> 요소가 삽입되거나 삭제되는 순간에 트리를 정리해야 하므로 트리 **크기에 비례하는 시간 복잡도**를 가지고 있습니다. 삽입 삭제가 너무 자주 일어나는 상황에서는 `TreeSet` 사용을 다시 한번 생각해 봐야 합니다.

`HashSet`과 `TreeSet`에서 1,000,000 번 `add()` 메소드를 수행한 결과 `HashSet`은 대략 200ms, `TreeSet`은 대략 700ms 가 소요됐습니다. 

# Iterator

`List`와 `Set`의 상위 인터페이스인 `Collection`은 `Iterable` 인터페이스를 상속받고 있습니다. 이 말은 `List`와 `Set`은 `Iterable` 인터페이스의 오퍼레이션인 `iterator()`를 구현해야 한다는 말이겠지요. `iterator()` 메소드는 `Iterator` 객체를 반환하는 메소드입니다.

![Collection, Iterable, Iterator 과 관계](/assets/2020-12-08-iterator-in-collection-framework/Untitled_1.png)

[리스트 4] Collection, Iterable, Iterator 과 관계

당연히 `ArrayList`, `HashSet` 모두 **`iterator()` 메소드를 호출해서 `Iterator` 객체를 가져올 수** 있습니다. 

`Iterator`는 컬렉션을 일관된 방법으로 바라볼 수 있도록 도와줍니다. 인덱스를 이용해 접근할 수 있는 `List`는 모르겠지만 `Set`은 컬렉션 내부 객체를 바라볼 방법이 없습니다. `Iterator`가 유일한 접근 방법입니다.

![인덱스를 사용해 직접 접근하는 List](/assets/2020-12-08-iterator-in-collection-framework/Untitled_2.png)

[리스트 5] 인덱스를 사용해 직접 접근하는 List

`Iterator`는 컬렉션에 접근하는 방법을 매우 단순하게 정의해놨습니다. 다음 것 있나? 다음 것 줘! 방금 가져온 것 삭제해! 이 3가지뿐입니다. 

![Iterator를 이용하여 접근하는 List](/assets/2020-12-08-iterator-in-collection-framework/Untitled_3.png)

[리스트 6] Iterator를 이용하여 접근하는 List

또 중요한 점은 `Iterator`는 컬렉션 전체 상태를 모니터링 하는 역할을 하게 할 수 있습니다. 

```java
List<Integer> integers = new ArrayList<>();
integers.add(1);
integers.add(2);
integers.add(3);
integers.add(4);

for (int i = 0; i < integers.size(); i++) {
    Integer integer = integers.get(i);
    System.out.println(integer);
}
```

위 코드를 한번 보겠습니다. 1에서 4까지 들어 있는 리스트를 하나씩 출력하는 코드입니다. 실행하면 아래와 같은 결과가 출력됩니다.

```
1
2
3
4
```

여기서 값이 2일 때 해당하는 값을 삭제하는 코드를 추가해 보겠습니다.

```java
List<Integer> integers = new ArrayList<>();
integers.add(1);
integers.add(2);
integers.add(3);
integers.add(4);

for (int i = 0; i < integers.size(); i++) {
    Integer integer = integers.get(i);
    System.out.println(integer);
    if (integer == 2)
        integers.remove(i);
}
```

이 코드의 출력값은 어떻게 될까요?

```
1
2
4
```

어떤가요? 이 출력이 기대했던 모습인가요?

리스트에 대한 직접 접근은 리스트 구조 변화에 대한 부작용이 발생합니다. 다음은 `Iterator`를 활용한 코드를 보시겠습니다.

```java
List<Integer> integers = new ArrayList<>();
integers.add(1);
integers.add(2);
integers.add(3);
integers.add(4);

Iterator<Integer> iterator = integers.iterator();
while(iterator.hasNext()){
    Integer integer = iterator.next();
    System.out.println(integer);
    if (integer == 2)
        iterator.remove();
}
```

```
1
2
3
4
```

중간에 값이 2인 경우 삭제를 하였지만, 출력에는 문제없이 1에서 4까지 되었습니다. `while` 문이 종료된 후에는 `integers` 리스트는 총 3개의 값만 가지고 있게 됩니다.

```java
System.out.println("----------");
iterator = integers.iterator();
while(iterator.hasNext()){
    Integer integer = iterator.next();
    System.out.println(integer);
}
```

```
1
2
3
4
----------
1
3
4
```

이렇게 `Iterator`를 이용하면 리스트의 변화에도 부작용이 데이터에 전파되지 않습니다. 그런데 코드를 보시면 `Iterator`의 `remove()` 메소드를 이용해서 리스트 데이터의 구조를 변화시키고 있습니다.  만약 `Iterator`가 아닌 `List`의 `remove()` 메소드나 `add()` 메소드를 사용했을 때 어떤 일이 생길까요?

```java
Set<String> stringSet = new HashSet<>();
stringSet.add("a");
stringSet.add("b");
stringSet.add("c");

Iterator<String> iterator = stringSet.iterator();
while(iterator.hasNext()){
    String next = iterator.next();
    System.out.println(next);
    if (next.equals("a")) {
        stringSet.remove(next);   //Set 의 메소드를 사용
    }
}
```

위 코드를 실행시키면 `ConcurrentModificationException`이 발생합니다. `Iterator`가 순회하고 있는데 다른 아이가 변경을 한 것이죠. 즉 **컬렉션을 순회 중일 때 새로운 아이템이 추가된다거나, 삭제되는 것을 감지하여 조기에 컨텍스트 오류를 막을 수 있습니다**.

> 문맥 오류는 가장 찾기 힘들고 예방하기 힘든 오류입니다.

# Iterable

앞선 코드를 보면 좀 불편한 구석이 없지 않아 있습니다. `List`에서 `Iterator`를 가져와야 하고 `while` 문으로 루프를 돌면서 새로운 값을 가져오는 구문도 필요합니다. 

![Collection, Iterable, Iterator 과 관계](/assets/2020-12-08-iterator-in-collection-framework/Untitled_1.png)

[리스트 12] Collection, Iterable, Iterator 과 관계

`Collection`과 `Iterable`, `Iterator`와의 관계를 보면 `Collection`은 `Iterable` 인터페이스를 상속하고 있는데 이 인터페이스를 구현하면 Java의 `foreach` 문에 사용할 수 있습니다.

루프 중간에 컬렉션의 구조를 변경하지 않고 순회만 한다면 다음과 같이 코드를 깔끔하게 줄일 수 있습니다.

```java
Set<String> stringSet = new HashSet<>();
stringSet.add("a");
stringSet.add("b");
stringSet.add("c");

for (String s : stringSet) {
    System.out.println(s);
}
```

# Helper 클래스들

Collection Framework에는 컬렉션을 사용할 때 편리함을 더해줄 유틸리티를 제공하고 있습니다. `Collections` 와 `Arrays` 클래스에서 주로 제공하고 있습니다.

## Collections

주로 컬렉션을 **둘러싸서 추가적인 기능하는 객체를 반환**하는 정적 메소드를 제공합니다. *변경 불가 컬렉션* 생성, *동기화 컬렉션* 생성, *강화된 타입 체크 컬렉션* 생성, *싱글톤 컬렉션 생성* 등을 지원합니다.

## Arrays

정렬, 검색, 초기화, 복사 등을 하는 정적 메소드를 제공합니다. 개인적으로 많이 쓰는 메소드는 `asList()` 인 데 

```java
List<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
```

와 같은 방식으로 편리하게 리스트를 만들어 낼 수 있습니다. 실 구현체는 `ArrayList`를 사용합니다.

# 다시보자 컬렉션 프레임워크

컬렉션 프레임워크는 개발할 때 상당히 많은 편리함과 이점을 줍니다. 우리가 필요로 하는 대부분 자료구조는 구현이 되어 있고, 성능 또한 최적화되어 있습니다. 또, 각기 다른 형태의 자료 구조로 되어 API가 서로 다름에도 불구하고 최소한의 노력으로 컬렉션을 사용할 수 있도록 잘 구조화되어 있습니다. 하지만 만들어져 있는 구현체의 특징을 잘 알고 사용하지 않으면 심각한 성능 저하나 찾기 힘든 오류, 동시성 문제 등에 부딪힐 수 있습니다. 

사용하기 전에 API 문서나 소스 코드를 한번 살펴보면 구현 원리와 주의점들을 쉽게 확인 할 수 있으니 꼭 사용 전에 둘러보시기 바랍니다. 소스 코드도 그다지 어렵지 않으니까 두려워하지 마세요!

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