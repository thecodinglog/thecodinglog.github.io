---
layout: post
title: 자바 제네릭 와일드 카드 사용 방법과 등장 배경
description: 자바 제네릭 와일드 카드 사용 방법과 등장 배경
date:   2020-12-15 09:00:00 +0900
author: Jeongjin Kim
categories: Java
tags:	Java Wildcard Generics Generic 
---
> _자바 제네릭의 와일드카드를 사용하는 방법과 제네릭 타입의 상위/하위타입에 관해 설명합니다._


지금까지 [제네릭 타입](/java/2020/12/09/java-generic-class.html)과 [제네릭 메소드](/java/2020/12/14/java-generic-method.html)를 선언하고 사용하는 방법에 대해 알아보았습니다. 하지만 제네릭은 생각처럼 만만한 개념이 아닙니다. 제네릭을 조금 더 깊이 이해하기 위해서는 주의해야 할 것들과 꼭 알아야 할 것들이 많습니다.

지금부터 제네릭 클래스의 상위/하위 타입에 대해서 알아보도록 하겠습니다.

# 서브 타입 일랑가 아닐랑가

`String` 타입의 값을 상위 타입인 `Object` 타입 변수에 넣을 수 있을까요?

```java
String string = "yaho";
Object object = string;
```

` `

당연히 넣을 수 있습니다. 그렇다면 이건 어떤가요?


`String` 타입 리스트 변수를 `Object` 타입 리스트 변수에 할당하는 구문입니다.

```java
List<String> stringList = new ArrayList<>();
List<Object> objectList = stringList;
```

` `


만약 이게 가능하다면 어떤 일이 일어날까요?

먼저 `objectList`에 `Integer` *123*을 넣어봅니다. `Object`이니까 당연히 가능하겠지요.

```java
List<String> stringList = new ArrayList<>();
List<Object> objectList = stringList;

objectList.add(123); // 추가된 구문
```

그러고 나서 `stringList`에서 방금 넣은 첫 번째 값을 가져옵니다. 이게 왜 가능하냐면 `objectList = stringList` 구분에서 서로 같은 참조 값을 가지기 때문에 `stringList`로도 `objectList`로 넣은 값을 가져올 수 있겠지요. 그리고 그걸 `String` 타입 변수에 넣어 봅니다.

```java
List<String> stringList = new ArrayList<>();
List<Object> objectList = stringList;
objectList.add(123);

String s = stringList.get(0);  // 추가된 구문
```

` `


어라? 우리의 전제 `List<Object> objectList = stringList;` 가 수행이 된다면, `String` 타입 변수에 `Integer` 타입 값을 넣는 게 가능해지는군요.

이건 말이 안 되지요? 그래서 `List<Object>` 타입 변수에 `List<String>` 타입 변수를 대입할 수 없다는 결론을 낼 수 있습니다. 

다시 말하면 `List<String>` 변수의 상위 타입이 `List<Object>`가 아니라는 것이지요! 

타입 파라미터 간에는 상/하위 관계가 없고 `raw-type` 간에만 상/하위 관계가 존재합니다.

` `

![](/assets/2020-12-15-java-generic-wildcard/2020-12-15-java-generic-wildcard_105337.png)

` `

이와 유사하게 컬렉션에 들어 있는 것들을 하나씩 꺼내서 출력하는 코드를 보겠습니다.
나름 재사용성에 신경 쓴다고 컬렉션의 어떤 타입이든 받아서 출력할 수 있도록 메소드 인자를 `Collection<Object>` 타입으로 한 메소드 `printCollection`을 만들었습니다.

```java
static void printCollection(Collection<Object> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}

public static void main(String[] args) {
    Collection<String> c = new ArrayList<>();
    c.add("hi");
    printCollection(c);
}
```

` `


그러나... 이 메소드가 의도대로 동작할까요? *컴파일조차 안 됩니다*. 왜요?

조금 전에  `List<Object>`가  `List<String>` 의 **상위타입이 아니라는 것과 같은 이유**입니다. 인자로 넘긴 컬렉션 타입은  `Collection<String>`이고 메소드 파라미터 타입은 `Collection<Object>`입니다. 당연히 하위 호환이 안 됩니다. 

# ? 등장

그러면 어떤 컬렉션이든지 받아서 출력하는 메소드를 만들면 어떻게 해야 할까요? 이걸 해결하기 위해 *wildcard*가 등장합니다. 간단히 `<Object>`를 `<?>`로 바꾸면 언제 에러 났냐는 듯 해결이 됩니다.

```java
static void printCollection(Collection<?> c) {
```

그럼 `Collection<Object>`의 상위타입이 `Collection<?>` 인가요? 그렇게 말하기는 좀 애매합니다. 왜냐하면 `?`은 타입은 아니니 까요^^; 그냥 `?`는 *그냥 임의의 어떤 것*으로 해석하는 게 쉬울 듯합니다. 상하 관계가 아닌 그냥 어떤 것!

자 그러면 여기서 *어떤 것*이기는 하는데 *`Number`를 상속한 어떤 것!* 이라고 하고 싶을 때는 어떻게 할까요? 지지난번 포스트에서 한정자 이야기를 했었는데 바로 그놈을 쓰면 됩니다. `extends`

위 코드를 아래처럼 고쳐 보겠습니다.

```java
static void printCollection(Collection<? extends Number> c) {
    for (Number e : c) {
        System.out.println(e);
    }
}

public static void main(String[] args) {
    Collection<Integer> c = new ArrayList<>();
    c.add(123);
    printCollection(c);
}
```

제네릭과 유사하게 `printCollection`의 인자 `c`는 *`Number` 타입 컬렉션 중 어떤 것*이기 때문에 `Number`가 할 수 있는 일을 할 수 있습니다.

# 목표 다시 해석해보기

이쯤에서 우리의 목표 메소드를 보면

```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col)
```

이제 저 `?` 가 어떤 존재인지 알게 되었습니다. 

`Collection<? extends T> col`에서 *T 타입보다 하위 타입인 어떤 것을 저장하는 컬렉션*을 인자로 받는 메소드인 것을 알 수 있습니다.

그럼 T 타입은 무엇이냐? 그러기 위해서는 조금 더 알아야 할 키워드가 있습니다. super.

이어지는 포스트는 이렇게나 복잡한 와일드카드에 대해 더 자세한 내용으로 이야기해 보겠습니다.

고고!



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