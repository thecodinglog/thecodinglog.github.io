---
layout: post
title: 자바 제네릭 메소드 선언 방법과 타입 추론
description: 자바 제네릭 메소드 선언 및 사용 방법, 타입추론으로 얻을 수 있는 이점
date:   2020-12-14 18:00:00 +0900
author: Jeongjin Kim
categories: Java
tags:	Java Generic-method Generics Generic 
---
> _자바 제네릭 메소드 선언 및 사용 방법, 타입추론으로 얻을 수 있는 이점을 설명합니다._


```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col){
}
```
<sub>[리스트 1] 우리의 목표</sub>

우리는 이 메소드를 한번 읽어 볼 꺼라고 이 고생을 하고 있습니다. 그런데 이 메소드를 정확하게 이해하기 위해서는 조금 더 힘을 내셔야 합니다. ^^;

[앞선 포스트](/java/2020/12/09/java-generic-class.html)에서는 클래스에 타입 파라미터를 추가해서 *String Box, Integer Box, Double Box* 등 원하는 데이터 타입을 넣을 수 있는 **제네릭 타입**을 만들었습니다. 

# 제네릭 메소드 정의하기

제네릭 타입은 한 개의 로직으로 여러 타입을 사용할 수 있도록 해 주는데요 이번에 할 **제네릭 메소드**는 **한 개의 로직으로 여러 타입의 인자와 리턴 타입을 가실 수 있는 메소드**입니다.

### 영향 범위(Scope)
제네릭 타입과 다르게 제네릭 메소드는 **타입 파라미터의 사용범위가 메소드 내로 한정**이 됩니다. 만약 제네릭 타입에 `T`라는 메소드 파라미터를 선언하더라도 제네릭 메소드에 타입 파라미터 `T`를 선언하면 제네릭 메소드의 타입 파라미터 `T`에 **우선순위**를 둡니다. *지역변수 선언한 것과 유사*하죠?

그럼 어떻게 정의하는지 모습을 한번 보겠습니다.

```java
public static <K, V> boolean methodName(K key, V value) {
    return true;
}
```
<sub>[리스트 2] 평범한 지네릭 메소드 선언 방법</sub>


우선 타입 파라미터의 리스트를 선언을 해야 하는데 `<K, V>`라고 되어있는 부분이 선언 부분입니다.  이 선언 부분은 반드시 **리턴 타입 앞**에 두어야 합니다.  이 메소드는 스태틱 타입의 메소드이고 타입 파라미터 `K`와 `V`를 쓰는 메소드가 되겠습니다. 타입 파라미터로 메소드의 인자 `key`, `value` 를 각각 받고 있네요.

# 제네릭 메소드 사용하기

이 메소드를 호출 해보겠습니다.

```java
boolean result = GenericMethods.<String, Integer>methodName("key", 3);
```
<sub>[리스트 3] 지네릭 메소드 호출 방법</sub>



스태틱 메소드이므로 메소드를 담고 있는 클래스(`GenericMethods`에 임의로 만들었습니다.)에 `methodName` 메소드를 호출을 했습니다. 보통 메소드와 다르게 메소드 이름 바로 앞에 `<String, Integer>`로 타입 인자(Type argument)를 주고 메소드의 인자로 `"key"`와 `3` 를 넣었습니다.

그럼 메소드 안에서는 넘겨받은 인자를 이용해서 로직을 수행하고 결과를 반환하게 되겠지요. 

### 타입 추론

그런데요, 메소드를 실행하는 과정이 좀 복잡해 보입니다. 평소에는 잘 사용하지 않은 타입 인자를 넣었기 때문인데, 사실 **여기 호출에서는 인자를 명시적으로 넣을 필요는 없습니다**.

```java
boolean result = GenericMethods.methodName("key", 3);
```
<sub>[리스트 4] 지네릭 메소드 간단하게 호출하는 방법</sub>


이렇게만 해줘도 잘 동작합니다.  왜냐하면 인자로 넘겨받은 `"key"`와 `3`으로 타입 파라미터의 타입을 **추론**할 수 있기 때문입니다. 

>`"key"`는 `String` 타입인걸 알 수 있고, 이 자리에 `K`가 있으니까 `K`는 `String`이 되겠군...

>`3`은 `Integer` 타입인걸 알 수 있고 이 자리에 `V`가 있으니까 `V`는 `Integer`가 되겠군...

이런 식이죠.

이렇게 생략을 하면 *static import*로 사용 가능해서 

```java
boolean result2 = methodName("key", 3);
```
<sub>[리스트 5] 짧게 축약된 호출 방법</sub>



이렇게 축약된 형태로 호출을 할 수 있습니다.

그래서 사실상 *대부분의 경우는 타입 인자를 명시하지 않기 때문에 제네릭 메소드인데 모르고 사용하는 경우가 많습니다.*

위에서는 메소드 파라미터에 넘겨진 인자들의 타입으로 타입 파라미터의 타입을 추론하였는데 **메소드의 리턴 값으로도 타입 파라미터의 값을 추론할 수 있습니다.**

```java
public static <T> List<T> emptyList() {
    return new ArrayList<T>();
}
```
<sub>[리스트 6] 빈 리스트를 반환하는 제네릭 메소드</sub>



[리스트 6]의 메소드는 `T`타입을 담을 수 있는 `ArrayList`를 반환하는 메소드입니다. 여기서 `T`는 어떻게 결정지을 수 있을까요?

```java
GenericMethods.<Integer>getList();
```
<sub>[리스트 7] 짤없는 제네릭 메소드 호출</sub>


제네릭 메소드를 호출하는 방법을 그대로 따르자면 [리스트 7]처럼 호출을 해야 하겠지요.

`<Integer>`같이 타입을 명시하는 구문을 영어로 **type witness**라고 하는데 한글로 고치면 타입 증인... 이라 좀 이상해서 그냥 타입 힌트 정도로 해석하는 게 나을 듯 합니다.(힌트도.. 영어긴 합니다만....)

 그러나 똑똑하게도 **반환하는 결과를 어디에 담느냐에 따라서 T 타입을 결정**짓게 할 수도 있습니다.

```java
List<Integer> list = GenericMethods.getList();
```
<sub>[리스트 8] 타입 추론을 이용한 제네릭 메소드 호출</sub>


반환 타입이 `List<Integer>` 인걸로 봐서 `T`는 `Integer` 이겠군... 이렇게 추론을 해서 타입을 결정할 수 있습니다.

그럼 언제 Type witness를 사용하느냐? 자바7을 포함하여 이전에는 메소드 인자 타입이 제네릭 타입 컬렉션인 경우에 제대로 추론을 하지 못하였는데 그 마저도 자바 8에서 개선이 되어 잘되고 있습니다. 그래서... 저는 한번도 타입 증인을 불러 본 적이 없기도하고 어느 상황에 꼭 명시를 해야 하는지 문서도 잘 없고 해서 솔직히 잘 모르겠습니다. 혹시 찾으면 이 문서를 업데이트 시켜 놓을께요!


# 목표 다시 해석해보기

이쯤에서 우리의 목표 메소드를 보면

```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col)
```
<sub>[리스트 9] 우리의 목표</sub>


이 정의는 제네릭 메소드 인 것을 알게 되었습니다. 그리고 이 메소드의 타입 파라미터는 `public` 다음에 오는 `<T extends Comparable<? super T>>` 복잡하지만, 이 구문인 걸 알 수 있습니다.

반환 타입은 `T`이고 메소드의 인자는 `Collection<? extends T>` 인건을 알 수 있겠네요.

이제 전체적인 구조는 파악이 되었고 이제 저 `?` 들만 남게 되었습니다.

[이어지는 포스트](/java/2020/12/15/java-generic-wildcard.html)에서는 `?` 와 앞선 포스트에서 본 한정자(Bounded Parameter)에 대해 알아보겠습니다.


혹시 제네릭 타입에 관해 기억이 나지 않으시면 [앞선 포스트](/java/2020/12/09/java-generic-class.html)를 참고해 주세요.

아디오스




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