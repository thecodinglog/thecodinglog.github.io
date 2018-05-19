---
layout: post
title: java 제네릭스
date:   2017-08-01 08:43:59
author: Jeongjin Kim
categories: JAVA
tags:	java generics
---
> 본 문서는 Oracle의 Java Document 중 [Generics ](https://docs.oracle.com/javase/tutorial/java/generics/index.html)부분을 발췌하여 번역한 문서입니다. 원래 의미를 정확하게 표현하기 위해, 필요한 경우 원문을 괄호 안에 표기하였습니다.
>
> This document is a translation of documents in Hangul excerpts of the Generics part of Oracle's Java Document. In order to understand the original meaning correctly, the original text is indicated in parentheses if necessary.

# 제네릭스(Generics)

버그 없는 소프트웨어란 존재하기 힘들다. 잘 계획하고 잘 개발해서 버그를 줄일 수는 있지만 없어지기란 사실상 힘들다. 특히 새로운 기능이 도입되거나, 코딩 라인수가 길어지게 되면 반드시 더 많이 발생한다.

그나마 다행인것은 버그 중에서도 쉽게 발견할 수 있는것도 존재한다는 것이다. 컴파일 타임버그가 그것인데 컴파일시 발생하는 에러 메시지 또는 경고 메시지는 버그를 일찍 발견해서 고칠 수 있게 한다.

그러나 런타임 버그는 그렇지 않다. 표면상에 드러나지도 않고 문제의 실제 원인은 다른 엉뚱한 곳이 있을 수도 있기 때문에 더욱 관리하기 힘든 버그이다.

제네렉스는 컴파일시 더 많은 버그를 탐지하여 코드 안전성을 확보할 수 있게 한다.

## 왜 제네릭스를 쓰는가?

제네릭스는 타입(Class, Interface)들을 정의할 때 타입을 파라미터로 쓸수 있게 하는 것이다.

일반 메소드와 마찬가지로 타입 파라미터는 코드의 재사용성을 높힌다. 메소드의 파라미터와 다른 점은 메소드의 파라미터는 값(Value)이고, 타입 파라미터의 파라미터는 타입(Type)이다.

#### 제네릭스가 주는 장점

* 컴파일 타임에 타입 체크
* 형변환 제거

```java
List list = new ArrayList();
list.add("hello");
String s = (String)list.get(0);

List<String> list = new ArrayList<String>();
list.add("hello");
String s = list.get(0);//no cast
```

* 개발자에에 알고리즘에 집중하도록 할 수 있음

## Generic Types

타입을 파라미터로 가지는 제네릭 클래스나 제네릭 인터페이스를 제네릭 타입이라고 한다. 일반 클래스인 Box Class를 어떻게 제네릭  클래스로 변경 할 수 있는지 아래 예를 보자.

#### 일반 Box Class

```java
public class Box {
    private Object object;
    public void set(Object object) { this.object = object; }
    public Object get() { return object; }
}
```

_set()_ 과 _get()_ 메소드는 _Object_ 타입을 파리미터로 받고 _Object_ 타입을 리턴하므로 어떤 타입이든 주고 받을 수 있다. 그러나 컴파일 타임에 어떤 클래스를 쓰고 있는지 검증할 방법이 없다.

코드에서 _Integer_ 를 넣고, 다른 코드 부분에서 _String_ 이라 생각하고 사용하게 되면 런타임 에러가 발생한다.

#### 제네릭 Box Class

제네릭 클래스는 아래와 같은 형태로 정의한다.

```java
class name<T1, T2, ..., Tn> { /* ... */ }
```

클래스 이름 뒤에 타입 파라미터 부분(&lt; &gt;)이 오는데, 이것이 타입 파라미터 (type parameters)또는 타입 변수(type variables) _T1_ ,  _T2_ , ..., _Tn_ 을 정의한다.

```java
/**
 * Generic version of the Box class.
 * @param <T> the type of the value being boxed
 */
public class Box<T> {
    // T stands for "Type"
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

코드에서 볼 수 있듯이 _Object_ 가 _T_ 로 교체됐다. 타입 변수는 기본(Primitive) 타입을 제외한 모든 타입을 쓸 수 있다.(Class, Interface 등)

#### 타입 파라미터 명명 규칙

타입 파라미터는 단일 영문 대문자로 명명한다. 일반적으로 알고있는 변수 명명 규칙과는 정 반대인데 그 이유는 이 규칙으로 하지 않으면 타입 파라미터와 일반 클래스 또는 인터페이스 이름의 차이를 구분하기가 어려울 수 있기 때문이다.

> **일반 적으로 쓰는 타입 파라미터 들**
>
> * E - Element (used extensively by the Java Collections Framework)
> * K - Key
> * N - Number
> * T - Type
> * V - Value
> * S,U,V etc. - 2nd, 3rd, 4th types

#### 제네릭 타입의 호출 및 객체화

제네릭 Box 클래스를 코드 안에서 참조하기 위해서는, T를 다른 고정 값으로 교체하는 작업인 제네릭 타입 호출(Generic type invocation)을 해야 한다. _Integer _ 를 예를 들면

```java
Box<Integer> integerBox;
```

제네릭 타입 호출이 일반 메서드 호출과 비슷하다고 생각할 수 있지만, 메서드에 인수를 전달하는 대신 _Integer_ 타입 인수를 Box 클래스 자체에 전달한다는 점이 다르다.

> **타입 파라미터(Type parameter)와 타입 인자(Type Argument) 용어**
>
> 많은 개발자들이 Type parameter와 Type argument을 구분없이 사용한다. 그러나 두 용어는 엄연히 다른 의미이다. 매개변수 타입을 만들기 위해 타입 인자를 제공한다. 그러므로 _Foo&lt;T&gt;_ 에서 _T_ 는 타입 파라미터이고 _Foo&lt;String&gt;_ 에서 _String_ 은 타입 인자이다.

일반적인 변수 선언과 비슷하게 위 코드도 Box 객체를 생성하지는 않고 _integerBox_ 는 Integer형의 Box의 참조 변수를 선언한 것이다.

제네릭 타입 호출를 일반적으로 파라미터와된 타입(Parameterized type)이라고 한다.

이 클래스를 초기화 하기 위해서는 _new_ 키워드를 사용해야 하며 클래스 이름과 양 괄호 사이에 _&lt;Integer&gt;_ 를 붙여야 한다.

#### 다이아몬드

Java SE 7 이상에서는 컴파일러가 컨텍스트의 타입 인수를 결정하거나 추론 할 수 있는 경우 타입 인자를 (&lt;&gt;)으로 바꿀 수 있다. 이 괄호 쌍 (&lt;&gt;)은 비공식적으로 다이아몬드라고 불린다. 예를 들어, 다음 구문으로 사용하여 _Box &lt;Integer&gt;_ 의 인스턴스를 만들 수 있다.

```java
Box<Integer> integerBox = new Box<>();
```

#### 다중 타입 파라미터

제네릭 클래스는 여러개의 타입 파라미터를 가질 수 있다. 예를 들어 제네릭 클래스인 _OrderdPair_ 클래스는 제네릭 인터페이스인 _Pair_ 인터페이스를 구현한다.

```java
public interface Pair<K, V> {
    public K getKey();
    public V getValue();
}

public class OrderedPair<K, V> implements Pair<K, V> {

    private K key;
    private V value;

    public OrderedPair(K key, V value) {
    this.key = key;
    this.value = value;
    }

    public K getKey()    { return key; }
    public V getValue() { return value; }
}
```

다음 두 구문은 두 개의 OrderdPair 클래스 객체를 생성한다.

```java
Pair<String, Integer> p1 = new OrderedPair<String, Integer>("Even", 8);
Pair<String, String>  p2 = new OrderedPair<String, String>("hello", "world");
```

_new OrderedPari&lt;String, Interger&gt;_ 구문은 _K_ 를 _String_ 으로, _V_ 을 _Integer_ 로 초기화한다. 그러므로 _OrderedPair_ 의 생성자 파라미터 타입은 _String_ 과 _Integer_ 이다. Autuboxing 기능 때문에 _String_ 과 _int_ 가 클래스로 전달된다.

다이아몬드 부분에서 언급했듯이 자바 컴파일러는 _K_ 와 _V_ 를 선언부 _OrderedPair&lt;String, Integer&gt;_ 에서 추론할 수 있으므로 위 구문은 아래와 같이 줄여 쓸 수 있다.

```java
OrderedPair<String, Integer> p1 = new OrderedPair<>("Even", 8);
OrderedPair<String, String>  p2 = new OrderedPair<>("hello", "world");
```

제네릭 인터페이스를 생성하려면 제네릭 클래스를 생성하는 방식을 따르면 된다.

#### 파리미터 타입(Parameterized Types)

타입 파라미터(예를 들면 _K_ 나 _V_ 같은 것)들은, 파라미터 타입(parameterized type) (예를 들어 _List&lt;String&gt;_ )으로도 다음과 같이 치환 가능하다.

```java
OrderedPair<String, Box<Integer>> p = new OrderedPair<>("primes", new Box<Integer>(...));
```

### 원시 타입(Raw Types)

제네릭 클래스나 인터페이스에서 타입 인자를 뺀 것을 원시타입(Raw Type)이라고 한다. 예를 들어 다음과 같이 제네릭 클래스 Box가 있을 때

```java
public class Box<T> {
    public void set(T t) { /* ... */ }
    // ...
}
```

_Box&lt;T&gt;_ 의 파라미터 타입을 만들기 위해서는 실 타입 인자(actual type argument)를 형식 타입 파라미터(formal type parameter) _T_ 에다가 넣어 줘야 한다.

```java
Box<Integer> intBox = new Box<>();
```

만약 실 타입 인자를 생략하면 Box&lt;T&gt;의 원시 타입을 생성하게 된다.

```java
Box rawBox = new Box();
```

그러므로 _Box_ 는 제네릭 타입 _Box&lt;T&gt;_ 의 원시 타입이다. 하지만 일반 클래스나 인터페이스는 원시 타입이 아니다.

원시 타입은 기존 코드에서 많이 보이는데 그 이유는 많은 Collections 같은 API 클래스들은 JDK 5.0 이전에 만들어 졌기 때문이다.

Raw type를 사용하면 제네릭 이전에 사용하던 것 처럼 쓸 수 있다. 하위 호환성을 위해 파리미터 타입(parameterized type)은 원시 타입에 할당할 수 있다.

```java
Box<String> stringBox = new Box<>();
Box rawBox = stringBox;               // OK
```

그러나 원시 타입을 파라미터 타입으로 할당 하려고 하면 경고가 발생한다.

```java
Box rawBox = new Box();           // rawBox is a raw type of Box<T>
Box<Integer> intBox = rawBox;     // warning: unchecked conversion
```

원사 타입으로 제네릭 타입에 정의된 메소드를 호출 할 때도 경고가 발생한다.

```java
Box<String> stringBox = new Box<>();
Box rawBox = stringBox;
rawBox.set(8);  // warning: unchecked invocation to set(T)
```

원시타입은 제네릭 타입 체크를 우회하여, 안전하지 않는 코드 발견을 런타임으로 미루게  되므로 사용을 지양해야 한다.

## Unchecked Error Messages

기존 코드와 제네릭 코드를 섞어 쓰게 되면 다음과 비 같은 경고 메시지를 종종 볼 수 있다.

```
Note: Example.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```

이 경고는 아래 샘플과 같은 코드를 작성하게 되면 발생하는데

```java
public class WarningDemo {
    public static void main(String[] args){
        Box<Integer> bi;
        bi = createBox();
    }

    static Box createBox(){
        return new Box();
    }
}
```

"unchecked"은 컴파일러가 타입 안전성을 보장하는데 필요한 모든 타입 검사를 수행하기에 충분한 타입정보를 갖고 있지 않음을 의미한다.

컴파일러에서 힌트는 주지만 'unchecked' 경고는 디폴트로 비활성화 되어있는데 이걸 모두 보려면 '-Xlint' 옵션을 주고 다시 컴파일 하면 된다.

## 제네릭 메소드

제네릭 메소드는 메소드 스스로 타입 파라미터를 도입해서 쓰는 메소드 이다. 제네릭 타입과 유사하지만 타입 파라미터의 범위(Scope)은 메소드로 제한된다. Static, non-static 메소드와 생성자도 지원된다.

제네릭 메서드는 꺾쇠 괄호 안에 파라미터가 오고 메서드의 반환 형식 앞에 둔다. 정적 제네릭 메소드의 경우, 타입 파라미터 섹션은 메소드의 리턴 유형 앞에 둔다.

_Util_ 클래스는 두 _Pair_ 객체를 비교하는 제네릭 메소드 _compare_ 를 가지고 있다.

```java
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
               p1.getValue().equals(p2.getValue());
    }
}

public class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey()   { return key; }
    public V getValue() { return value; }
}
```

아래는 메소드 호출을 위한 완전한 구분은 이다.

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean same = Util.<Integer, String>compare(p1, p2);
```

그런데 컴파일러가 타입을 추론할 수 있기 때문에, 보통은 타입을 명시적으로 적지 않고 생략한다.

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean same = Util.compare(p1, p2);
```

타입 추론은 제네릭 메소드를 일반 메소드인것처럼 쓸 수 있게 한다.

## 경계가 있는 타입 파라미터(Bounded Type Parameters)

파라미터 타입(Parameterized Type)에서 사용할 수 있는 타입 인자를 제한하고자 할 때가 있다. 예를 들어 숫자를 처리하는 메소드는 _Number_ 나 _Number_ 의 하위 클래스의 인스턴스만 허용하고 하는 경우이다. 이 때문에 경계가 있는 타입 파라미터(bounded type parameter)[^1]를 사용한다.

Bounded type parameter를 선언하려면 타입 파리미터명 다음 _extends_ 키워드와 상위 타입(위 예제인 경우 _Number_ )을 차례로 나열하면 된다. 여기서 _extends_ 는 _extends(클래스_에서) 또는 _implements_ (인터페이스에서)를 의미한다.

```java
public class Box<T> {

    private T t;          

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }

    public <U extends Number> void inspect(U u){
        System.out.println("T: " + t.getClass().getName());
        System.out.println("U: " + u.getClass().getName());
    }

    public static void main(String[] args) {
        Box<Integer> integerBox = new Box<Integer>();
        integerBox.set(new Integer(10));
        integerBox.inspect("some text"); // error: this is still String!
    }
}
```

Bounded type parameter를 넣기 위해 _inspect_ 메소드를 수정하면 컴파일 오류가 발생한다. 왜냐하면 _inspect_ 메소드를 _String_ 타입의 인자를 넘겨 호출하고 있기 때문에다.

```
Box.java:21: <U>inspect(U) in Box<java.lang.Integer> cannot
  be applied to (java.lang.String)
                        integerBox.inspect("10");
                                  ^
1 error
```

이렇게 Bounded type parameter는 사용할 수 있는 타입에 제약을 줄 수 있을 뿐만 아니라, Bound된 타입의 메소드를 사용할 수 있게 된다.

```java
public class NaturalNumber<T extends Integer> {

    private T n;

    public NaturalNumber(T n)  { this.n = n; }

    public boolean isEven() {
        return n.intValue() % 2 == 0;
    }

    // ...
}
```

_isEven_ 메소드는 _n_ 을 통해 _Integer_ 클래스에 정의 된 _intValue_ 메소드를 호출한다.

#### 다중 경계(Multiple Bounds)

앞의 예제는 단일 바인딩에서 타입 파라미터를 사용하는 방법을 보여 주지만 타입 파라미터는 여러 경계를 가질 수 있다.

```java
<T extends B1 & B2 & B3>
```

다중 경계로 선언된 타입 변수는 열거된 타입들의 서브 타입이고 만약 경계 중 하나가 클래스인 경우 먼저 지정해야 한다.

```java
Class A { /* ... */ }
interface B { /* ... */ }
interface C { /* ... */ }

class D <T extends A & B & C> { /* ... */ }
```

경계 A가 먼저 오지 않는다면 컴파일 에러가 발생한다.

```
class D <T extends B & A & C> { /* ... */ }  // compile-time error
```

### 제네릭 메소드와 Bounded Type Parameters

Bounded type parameter는 제네릭 알고리즘 구현의 핵심이다. 배열 _T \[ \]_ 에서 지정된 요소 _elem_ 보다 큰 요소의 수를 계산하는 다음 메소드를 보면

```java
public static <T> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e > elem)  // compiler error
            ++count;
    return count;
}
```

이 메서드의 구현은 간단하지만 greater than 연산자 (&gt;)는 _short, int, double, long, float, byte_ 및 _char_ 같은 기본 유형에만 적용되므로 컴파일되지 않는다. &gt; 연산자를 사용하여 객체를 비교할 수 없기 때문에  _Comparable &lt;T&gt;_ 인터페이스로 경계가 타입 파라미터를 사용하면 된다.

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

적용 결과는

```java
public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e.compareTo(elem) > 0)
            ++count;
    return count;
}
```

## 제네릭스, 상속, 하위 타입(Generics, Inheritance, Subtypes)

특정 타입의 오브젝트를 이 타입과 호환이 되는 다른 타입에 할당하는 것은 가능하다. 예를 들어 _Object_ 는 _Integer_ 의 Supertype 중 하나기 떄문에 _Integer_ 값을 _Object_ 에 할당할 수 있다.

```java
Object someObject = new Object();
Integer someInteger = new Integer(10);
someObject = someInteger;   // OK
```

객체 지향에서 이것을 "is a" 관계라고 한다. _Integer_ 는 일종의 _Object_ 이므로 할당이 허용된다. 그러나 _Integer_ 는 일종의 _Number_ 이기도하므로 다음 코드도 유효하다.

```java
public void someMethod(Number n) { /* ... */ }

someMethod(new Integer(10));   // OK
someMethod(new Double(10.1));   // OK
```

제네릭에서도 마찬가지로 타입 인자로 _Number_ 를 넘겨서 제네릭 타입호출을 하면, _Number_ 와 호환되는 모든 _add_ 메소드 호출도 허용된다.

```java
Box<Number> box = new Box<Number>();
box.add(new Integer(10));   // OK
box.add(new Double(10.1));  // OK
```

다음 아래의 메소드에서

```java
public void boxTest(Box<Number> n) { /* ... */ }
```

어떤 타입이 인자로 허용이 될까? 이 메소드의 시그너처를 보면 _Box&lt;Number&gt;_ 유형의 단일 인수를 받는다는 것을 알 수 있다. 이게 무슨 의미인가? _Box&lt;Interger&gt;_ 나 _Box&lt;Double&gt;_  어용한다는 의미인가? 답은 "아니오"이다. 왜냐하면 _Box&lt;Integer&gt;_ 와 _Box&lt;Double&gt;_ 은 _Box&lt;Number&gt;_ 의 서브타입이 아니기 때문이다.

이건은 제네릭으로 프로그래밍 할 때 일반적으로 자주하는 오해이다. 하지만 이것은 반드시 알아야하는 중요한 컨셉이다.

![](https://docs.oracle.com/javase/tutorial/figures/java/generics-subtypeRelationship.gif "diagram showing that Box&amp;lt;Integer&amp;gt; is not a subtype of Box&amp;lt;Number&amp;gt;")

_Integer_ 는 _Number_ 의 서브타입 이지만, _Box&lt;Integer&gt;_ 는  _Box&lt;Number&gt;_ 의 서브타입이 아니다.

> **Note : **두 타입 A, B가 있을 때(예를 들면 _Number_ 와 _Integer_ ), _MyClass&lt;A&gt;_ 와 _MyClass&lt;B&gt;_ 는 A와 B가 연관 관계가 있더라도 아무런 관련이 없다. 두 클래스의 공통 부모는 _Object_ 뿐이다.

#### 제네릭 클래스와 하위 타입

제네릭 클래스나 인터페이스를 확장하거나 구현해서 서브타입을 만들 수 있다. 어떤 클래스/인터페이스의 타입 파라미터와 다른 클래스/인터페이스의 타입 파라미터 간의 관계는 _extends_와 _implements _로 결정된다.

_Collection _클래스를 예로 들자면 _ArrayList&lt;E&gt;_는 _List &lt;E&gt;_를 구현하고 _List&lt;E&gt;_는 _Collection&lt;E&gt;_를 확장한다. 따라서 _ArrayList&lt;String&gt;_은 _List&lt;String&gt;_의 하위 유형이며 _Collection&lt;String&gt;_의 하위 유형이다. 타입 인자를 변경하지 않는 한 유형간에 하위 타입관계가 유지된다.

![](https://docs.oracle.com/javase/tutorial/figures/java/generics-sampleHierarchy.gif "diagram showing a sample collections hierarchy: ArrayList&amp;lt;String&amp;gt; is a subtype of List&amp;lt;String&amp;gt;, which is a subtype of Collection&amp;lt;String&amp;gt;.")

Collections 계층 구조의 예

새로운 List 인터페이스를 구현하고 제네릭 타입 P를 추가적으로 선언된 _PayloadList _클래스를 작성하면 아래와 같이 될 것이다.

```java
interface PayloadList<E,P> extends List<E> {
  void setPayload(int index, P val);
  ...
}
```

아래 선언된 목록은_ List&lt;String&gt;_의 하위타입이 될 수 있는 _PayloadList _선언부이다.

* PayloadList&lt;String, String&gt;
* PayloadList&lt;String, Integer&gt;
* PayloadList&lt;String, Exception&gt;

![](https://docs.oracle.com/javase/tutorial/figures/java/generics-payloadListHierarchy.gif "diagram showing an example PayLoadList hierarchy: PayloadList&amp;lt;String, String&amp;gt; is a subtype of List&amp;lt;String&amp;gt;, which is a subtype of Collection&amp;lt;String&amp;gt;. At the same level of PayloadList&amp;lt;String,String&amp;gt; is PayloadList&amp;lt;String, Integer&amp;gt; and PayloadList&amp;lt;String, Exceptions&amp;gt;.")

PayloadList 계층 구조의 예

## 타입 추론

타입 추론은 자바 컴파일러가 각각의 메소드 호출과 대응하는 선언을 보고 호출 할 수있는 타입 인자를 결정하는 기능이다. 추론 알고리즘은 인수의 유형과, 가능하면 결과가 할당되거나 반환되는 유형을 결정한다. 최종적으로 추론 알고리즘은 모든 인수와 함께 호환되는 가장 구체적인 유형을 찾는다.

```java
static <T> T pick(T a1, T a2) { return a2; }
Serializable s = pick("d", new ArrayList<String>());
```

#### 타입 추론과 제네릭 메소드

제네릭 메소드를 통해 타입 추론을 쉽게 볼 수 있다. 제네릭 메소드는 타입 추론을 통해 타입을 명시하지 않고 일반 메소드처럼 호출 할 수 있게 한다.

```java
public class BoxDemo {

  public static <U> void addBox(U u,
      java.util.List<Box<U>> boxes) {
    Box<U> box = new Box<>();
    box.set(u);
    boxes.add(box);
  }

  public static <U> void outputBoxes(java.util.List<Box<U>> boxes) {
    int counter = 0;
    for (Box<U> box: boxes) {
      U boxContents = box.get();
      System.out.println("Box #" + counter + " contains [" +
             boxContents.toString() + "]");
      counter++;
    }
  }

  public static void main(String[] args) {
    java.util.ArrayList<Box<Integer>> listOfIntegerBoxes =
      new java.util.ArrayList<>();
    BoxDemo.<Integer>addBox(Integer.valueOf(10), listOfIntegerBoxes);
    BoxDemo.addBox(Integer.valueOf(20), listOfIntegerBoxes);
    BoxDemo.addBox(Integer.valueOf(30), listOfIntegerBoxes);
    BoxDemo.outputBoxes(listOfIntegerBoxes);
  }
}
```

이 코드의 결과는

```
Box #0 contains [10]
Box #1 contains [20]
Box #2 contains [30]
```

제네릭 메소드 _addBox_는 타입 파리미터 _U_를 정의했다. 일반적으로 자바 컴파일러는 제네릭 메소드 호출의 타입 파라미터를 추론할 수 있다. 따라서 대부분의 경우는 지정할 필요가 없다. _addBox_라는 제네릭 메소드를 호출할 때 타입 파라미터의 타입 힌트(type witness)를 아래와 같이 줄 수 있다.

```java
BoxDemo.<Integer>addBox(Integer.valueOf(10), listOfIntegerBoxes);
```

타입 힌트를 생략하면 자바 컴파일러는 자동적으로 메소드의 인자를 보고 타입 파라미터가 _Integer_라고 추론한다.

#### 타입 추론과 제네릭 클래스의 객체화

컴파일러가 컨텍스트의 형식 인수를 유추 할 수있는 경우 제네릭 클래스의 생성자를 호출하는 데 필요한 형식 인수를 형식 매개 변수 집합 (&lt;&gt;)의 빈 집합으로 바꿀 수 있다. 이 괄호 쌍은 비공식적으로 다이아몬드라고 부른다.

```java
Map<String, List<String>> myMap = new HashMap<String, List<String>>();
```

생성자의 파라미터 타입(paramterized type)를 빈 타입 파리미터로 바꿀 수 있다.

```java
Map<String, List<String>> myMap = new HashMap<>();
```

제네릭 클래스를 인스턴스화 할 때 타입 추론을 사용하려면 다이아몬드를 꼭 써야한다. 다음 예제에서는 _HashMap()_ 생성자가 _Map &lt;String, List &lt;String &gt;&gt; _형식이 아닌 _HashMap _원시 타입(raw type)을 반환하기 때문에 컴파일러에서 확인되지 않은 변환 경고를 생성한다.

```java
Map<String, List<String>> myMap = new HashMap(); // unchecked conversion warning
```

#### 제네릭 클래스와 일반 클래스의 제네릭 생성자와 타입 추론

생성자는 제네릭 클래스든 일반 클래스든 제네릭이 될 수 있다. 다시 말하면 생성자 선언할 때 생성자의 형식 타입 파라미터를 가질 수 있다는 의미이다.

```java
class MyClass<X> {
  <T> MyClass(T t) {
    // ...
  }
}
```

MyClass 클래스의 객체화 부분을 보면

```java
new MyClass<Integer>("")
```

이 구문은 파라미터 타입(parameterized type)인 _MyClass&lt;Integer&gt;_의 객체를 만든다. 여기서 제네릭 클래스인 _MyClass&lt;X&gt;_ 의 형식 파라미터 _X_에 대하여 _Integer _타입이라고 명시하고 있다. 형식 타입 파라미터 _T_를 가지고 있는 생성자를 보면 컴파일러가 형식 타입 파라미터 _T _를 _String _타입이라고 유추하는데 그 이유는 생성자의 실제 파라미터(actual parameter)가  " " 로 _String_ 객체이기 때문이다.

#### 대상 타입(Target Types)

자바 컴파일러는 제네릭 메소드 호출(generic method invocation)의 타입 파라미터를 유추하기 위해 대상 타입의 정보를 이용한다. 표현식의 대상 타입은 자바 컴파일러가 그 표현식이 어디에 나타나는지에 따라 기대하는 데이터 타입이다. 다음 메소드의 Collections.emptyList 메소드를 보면

```java
static <T> List<T> emptyList();
```




[^1]: 적절한 한글명이 없어 이하 원문으로 표기함
