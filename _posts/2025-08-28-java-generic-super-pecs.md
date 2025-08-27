---
layout: post
title: 자바 제네릭 super 와일드카드와 PECS 원칙
description: 자바 제네릭 super 와일드카드와 PECS 원칙
date:   2025-08-28 15:40:00 +0900
author: Jeongjin Kim
categories: Java
tags:	Java Wildcard Generics Generic 
---

지금까지 [제네릭 클래스](/java/2020/12/09/java-generic-class.html), [제네릭 메소드](/java/2020/12/14/java-generic-method.html), [와일드카드의 extends](/java/2020/12/15/java-generic-wildcard.html)에 대해서 알아보았습니다. 드디어 우리의 목표 메소드를 완전히 해석할 수 있는 마지막 퍼즐 조각인 `super` 키워드에 대해 알아보겠습니다.

```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col)
```
<sub>[리스트 1] 드디어 완전히 해석할 목표</sub>

## super 와일드카드의 필요성

앞선 포스트에서 `extends` 와일드카드를 배웠는데, 왜 `super`라는 또 다른 와일드카드가 필요할까요? 간단한 예제를 통해 알아보겠습니다.

숫자들을 담을 수 있는 리스트에 숫자를 추가하는 메소드를 만들어 보겠습니다.

```java
static void addNumbers(List<Number> list) {
    list.add(1);
    list.add(2.0);
    list.add(3.14f);
}
```
<sub>[리스트 2] Number 타입 리스트에 숫자를 추가하는 메소드</sub>

이 메소드를 사용해보겠습니다.

```java
public static void main(String[] args) {
    List<Number> numberList = new ArrayList<>();
    addNumbers(numberList);
    System.out.println(numberList); // [1, 2.0, 3.14]
}
```
<sub>[리스트 3] Number 리스트로 잘 동작하는 코드</sub>

잘 동작합니다. 그런데 만약 `Object` 타입 리스트에도 숫자를 추가하고 싶다면 어떻게 해야 할까요?

```java
public static void main(String[] args) {
    List<Object> objectList = new ArrayList<>();
    addNumbers(objectList); // 컴파일 에러!
}
```
<sub>[리스트 4] Object 리스트로는 동작하지 않는 코드</sub>

컴파일 에러가 발생합니다. `List<Object>`는 `List<Number>`의 상위 타입이 아니기 때문입니다. 

그렇다면 `extends` 와일드카드를 사용하면 어떨까요?

```java
static void addNumbers(List<? extends Number> list) {
    list.add(1); // 컴파일 에러!
}
```
<sub>[리스트 5] extends 와일드카드로는 add가 안되는 코드</sub>

이것도 안됩니다! `extends` 와일드카드는 해당 타입이나 하위 타입을 **읽기**는 할 수 있지만, **쓰기**(추가)는 할 수 없습니다. 왜냐하면 `List<? extends Number>`가 실제로 `List<Integer>`일 수도 있는데, 여기에 `Double` 타입을 넣으면 문제가 되기 때문입니다.

## super 와일드카드 등장

이럴 때 사용하는 것이 바로 `super` 와일드카드입니다.

```java
static void addNumbers(List<? super Number> list) {
    list.add(1);        // Integer -> Number로 업캐스팅, OK
    list.add(2.0);      // Double -> Number로 업캐스팅, OK  
    list.add(3.14f);    // Float -> Number로 업캐스팅, OK
}
```
<sub>[리스트 6] super 와일드카드를 사용한 메소드</sub>

이제 이 메소드를 다양한 타입의 리스트와 함께 사용할 수 있습니다.

```java
public static void main(String[] args) {
    List<Number> numberList = new ArrayList<>();
    List<Object> objectList = new ArrayList<>();
    
    addNumbers(numberList);  // OK
    addNumbers(objectList);  // OK
    
    System.out.println(numberList); // [1, 2.0, 3.14]
    System.out.println(objectList); // [1, 2.0, 3.14]
}
```
<sub>[리스트 7] super 와일드카드로 해결된 코드</sub>

`List<? super Number>`는 `Number`의 상위 타입인 어떤 것을 담는 리스트라는 의미입니다. `Number`는 `Object`의 하위 타입이므로, `Number`와 그 하위 타입들(`Integer`, `Double`, `Float` 등)을 `Object` 리스트에 넣는 것이 안전합니다.

## PECS 원칙 (Producer Extends Consumer Super)

제네릭을 사용할 때 언제 `extends`를 쓰고 언제 `super`를 써야 할지 헷갈릴 수 있습니다. 이를 위해 **PECS 원칙**이라는 유용한 가이드라인이 있습니다.

- **Producer Extends**: 데이터를 **생산(읽기)**하는 경우 → `? extends T`
- **Consumer Super**: 데이터를 **소비(쓰기)**하는 경우 → `? super T`

### Producer Extends 예제

```java
// 리스트에서 데이터를 읽어서 처리하는 메소드 (Producer)
static double sum(List<? extends Number> numbers) {
    double result = 0;
    for (Number num : numbers) {  // 읽기만 함
        result += num.doubleValue();
    }
    return result;
}

public static void main(String[] args) {
    List<Integer> intList = Arrays.asList(1, 2, 3);
    List<Double> doubleList = Arrays.asList(1.1, 2.2, 3.3);
    
    System.out.println(sum(intList));    // 6.0
    System.out.println(sum(doubleList)); // 6.6
}
```
<sub>[리스트 8] Producer 역할을 하는 extends 와일드카드</sub>

### Consumer Super 예제

```java
// 리스트에 데이터를 추가하는 메소드 (Consumer)
static void addIntegers(List<? super Integer> list) {
    list.add(10);  // 쓰기만 함
    list.add(20);
    list.add(30);
}

public static void main(String[] args) {
    List<Integer> intList = new ArrayList<>();
    List<Number> numList = new ArrayList<>();
    List<Object> objList = new ArrayList<>();
    
    addIntegers(intList);  // OK
    addIntegers(numList);  // OK  
    addIntegers(objList);  // OK
}
```
<sub>[리스트 9] Consumer 역할을 하는 super 와일드카드</sub>

## 실제 Collections.copy() 메소드 살펴보기

실제 Java API에서 PECS 원칙이 어떻게 적용되는지 `Collections.copy()` 메소드를 보겠습니다.

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    // dest 리스트에 src 리스트의 요소들을 복사
}
```
<sub>[리스트 10] Collections.copy() 메소드 시그니처</sub>

- `src`는 **Producer** 역할: 데이터를 읽어오므로 `? extends T`
- `dest`는 **Consumer** 역할: 데이터를 받아 저장하므로 `? super T`

이렇게 설계함으로써 다음과 같은 유연성을 얻을 수 있습니다.

```java
List<Integer> integerList = Arrays.asList(1, 2, 3);
List<Number> numberList = new ArrayList<>();
List<Object> objectList = new ArrayList<>();

Collections.copy(numberList, integerList);  // Integer -> Number
Collections.copy(objectList, integerList);  // Integer -> Object
```
<sub>[리스트 11] Collections.copy()의 유연한 사용 예제</sub>

## 목표 메소드 완전 해석하기

이제 드디어 우리의 목표 메소드를 완전히 해석할 수 있습니다!

```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col)
```

하나씩 분해해보겠습니다:

1. **`<T extends Comparable<? super T>>`**: 
   - `T`는 `Comparable`을 구현한 타입이어야 함
   - `Comparable<? super T>`는 T 타입 또는 T의 상위 타입을 비교할 수 있어야 함
   - 예: `String`은 `Comparable<String>`을 구현하므로 가능

2. **`Collection<? extends T> col`**:
   - 컬렉션에서 데이터를 **읽어와서**(Producer) 비교할 것이므로 `extends` 사용
   - T 타입 또는 T의 하위 타입을 담은 컬렉션을 받을 수 있음

3. **`T max(...)`**:
   - 결과로 T 타입을 반환

## 왜 Comparable<? super T>일까?

마지막으로 `Comparable<? super T>`가 왜 필요한지 알아보겠습니다.

```java
class Fruit implements Comparable<Fruit> {
    private String name;
    
    public Fruit(String name) { this.name = name; }
    
    @Override
    public int compareTo(Fruit other) {
        return this.name.compareTo(other.name);
    }
    
    @Override
    public String toString() { return name; }
}

class Apple extends Fruit {
    public Apple() { super("Apple"); }
}

class Orange extends Fruit {
    public Orange() { super("Orange"); }
}
```
<sub>[리스트 12] Fruit 클래스와 하위 클래스들</sub>

만약 `Comparable<T>`만 사용한다면:

```java
// 이렇게 선언했다면
public static <T extends Comparable<T>> T max(Collection<? extends T> col)

// Apple 리스트의 최댓값을 구할 때 문제 발생
List<Apple> apples = Arrays.asList(new Apple(), new Apple());
Apple maxApple = max(apples); // 컴파일 에러!
```
<sub>[리스트 13] `Comparable<T>`만 사용할 때의 문제점</sub>

`Apple`은 `Comparable<Fruit>`를 상속받으므로 `Comparable<Apple>`을 직접 구현하지 않기 때문에 에러가 발생합니다.

하지만 `Comparable<? super T>`를 사용하면:

```java
public static <T extends Comparable<? super T>> T max(Collection<? extends T> col)

List<Apple> apples = Arrays.asList(new Apple(), new Apple());
Apple maxApple = max(apples); // OK!
```
<sub>[리스트 14] super 와일드카드로 해결된 코드</sub>

`Apple`이 `Comparable<Fruit>`를 구현하고, `Fruit`는 `Apple`의 상위 타입이므로 조건을 만족합니다.

## 마무리

이제 복잡해 보였던 제네릭 메소드를 완전히 이해할 수 있게 되었습니다:

```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col)
```

- 컬렉션에서 요소들을 읽어와 비교하여 최댓값을 찾는 메소드
- T 타입은 자신 또는 상위 타입과 비교 가능해야 함
- 다양한 타입의 컬렉션에서 유연하게 동작

PECS 원칙을 기억하세요:
- **Producer Extends**: 읽기 → `? extends T`
- **Consumer Super**: 쓰기 → `? super T`

이 원칙만 기억하면 복잡한 제네릭도 더 쉽게 이해하고 설계할 수 있습니다!

이전 포스트 [제네릭 클래스](/java/2020/12/09/java-generic-class.html), [제네릭 메소드](/java/2020/12/14/java-generic-method.html), [와일드카드의 extends](/java/2020/12/15/java-generic-wildcard.html) 를 한번 더 찬찬히 읽어보시고 어렵게 느껴지는 자바 제네릭을 완벽하게 이해하셨으면 합니다.



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