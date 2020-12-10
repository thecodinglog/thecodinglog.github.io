---
layout: post
title: 자바 제네릭 타입과 extend 키워드로 경계 설정하기
description: 자바 제네릭 클래스는 왜 사용하는가?
date:   2020-12-09 18:00:00 +0900
author: Jeongjin Kim
categories: Java
tags:	Java Generics Generic
---
> _이 포스트는 자바 제네릭을 사용하는 이유와 제네릭 클래스 선언 방법, 범위 설정하는 방법을 설명합니다._





```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col)
```

위 메소드 타입은 `Collections`의 `max()` 메소드를 약간 정리한 것입니다. 쉽게 읽히시나요? 이 정도쯤은 읽을 수 있다! 라고 하시는 분은 포스트를 읽을 필요가 없습니다. 지금부터 위 메소드를 어떻게 해석해야 할지 알아보기 위해서 차근차근 필요한 사항을 알아보도록 하겠습니다. 


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




# 간단한 박스 때기 클래스

아무 데이터를 담을 수 있는 `SimpleBox` 클래스를 하나 작성해 보겠습니다.

```java
public class SimpleBox {
    private Object data;

    public Object getData() { return data; }

    public void setData(Object data) { this.data = data; }
}
```

이 박스에 문자열을 하나 넣고 사용하는 코드를 만들어 봅니다.

```java
SimpleBox simpleBox = new SimpleBox();
simpleBox.setData("내 문자열");
```

*"내 문자열"* 이라는 데이터를 넣은 박스를 사용하는 코드를 만들어 봅니다.

```java
Integer data = (Integer) simpleBox.getData();
int i = data + 1;
System.out.println(i);
```

`simpleBox`에 있는 데이터를 가져온 뒤 *+1* 연산을 수행해서 출력하는 구문입니다. 이렇게 작성해서 컴파일하면 컴파일이 될까요?



잘 됩니다. 문법상 아무런 문제가 없기 때문에 컴파일러는 잡아내지 못합니다. 



그럼 실행을 해보면 잘 되나요?

잘 안됩니다. `ClassCastException` 이 발생합니다. `(Integer) simpleBox.getData();` 에서 `getData()` 메소드로 가져온 데이터의 실제 타입은 `String`인데 `Ingeter`로 형 변환을 하려고 했기 때문입니다.

이렇게 다양한 타입을 받을 수 있는 클래스는 **컴파일 타임에 오류를 발견할 수가 없기 때문**에 **시스템의 안정성이 급격히 떨어집**니다. 그러면 타입을 명확히 지정하면 되지 않을까요? 그렇게 한번 다시 만들어 보겠습니다.

```java
public class StringBox {
    private String data;

    public String getData() { return data; }

    public void setData(String data) { this.data = data; }
}
```

`String` 타입만 넣을 수 있는 `StringBox`를 만들었습니다. 이제 사용하는 코드를 작성해 보겠습니다.

```java
StringBox stringBox = new StringBox();
stringBox.setData("내 문자열");

Integer data = (Integer) stringBox.getData();
int i = data + 1;
System.out.println(i);
```

[리스트 3]의 코드처럼 Box에 있는 데이터를 가져와 `Integer`로 형 변환 뒤 연산하는 코드입니다. 컴파일되나요?

됩니다. 당연하게도 이 코드에서는 **컴파일러가 타입을 명확히 알고 있기 때문에 형 변환 오류를 미리 발견**합니다. 이렇게 하면 프로그램의 안정성이 이전 코드보다 매우 좋아집니다.

음.. 그런데 `Integer` 타입을 담을 수 있는 Box도 필요한 일이 생겼습니다. 그래서 `IngeterBox`를 만들기로 했습니다.

```java
public class IntegerBox {
    private Integer data;

    public Integer getData() { return data; }

    public void setData(Integer data) { this.data = data; }
}

```

만들고 나서 보니 `CharBox`, `LongBox` 등등 만들어야 할 일이 생겼습니다. 각각 다 만드는 게 좋은 방법일까요?

아무래도 좋은 방법이라고 말하기는 힘들 것 같습니다. 왜냐하면 타입 말고 다른 로직(여기는 getter, setter밖에 없습니다만...)은 모두 같기 때문에 **코드 중복이 심각하게 발생**하기 때문입니다.

**컴파일 타임에 형에 대한 안정성을 확보하고, 코드 중복을 해결**하는 방법이 바로 **Generics** 입니다.

# 제네릭스 박스 때기 클래스

Generics를 이용해서 Box를 새로 만들어 보겠습니다.

```java
public class GenericBox<T> {
    private T data;

    public T getData() { return data; }

    public void setData(T data) { this.data = data; }
}
```

앞선 Box 구현과 다르게 `<T>` 라는 표시가 생겼고 타입을 선언하던 자리에는 `T` 라는 글자가 대체하고 있습니다. 딱 보면 아시겠지만, `T`는 `String`, `Integer`, `Char` 등 Box 클래스를 만들 때 받은 타입 정보와 교체되어서 실 객체가 만들어집니다.

여기서 `T` 를 ***타입 파라미터(Type parameter)***라고 합니다.

그럼 이 Box를 사용해 보도록 하겠습니다.

```java
GenericBox<String> genericBox = new GenericBox<>();
genericBox.setData("내 문자열");
```

`GenericBox` 뒤에 `<String>`을 넣어 이 박스는 `String`을 위한 박스라고 지정합니다. `StringBox`와 마찬가지로 `new` 키워드를 이용해서 객체를 생성합니다. 

위 코드에서 `<String>`의  `String`을 ***타입 인자(Type argument)*** 라고 합니다.

약간 **메서드 호출과 비슷**하게 이해하셔도 좋을 듯합니다. *"`GenericBox` 라는 메소드에 `String` 이라는 인자를 넘겨서 `String` 타입의 `GenericBox`를 리턴받는다"* 라는 느낌으로요.

위 코드에서 [리스트 3] 코드처럼 사용하는 코드를 추가해보겠습니다.

```java
Integer data = (Integer) genericBox.getData();
int i = data + 1;
System.out.println(i);
```

컴파일이 되나요?

안됩니다. 이렇게 타입 체크도 하고 코드의 재사용성도 높이는 두 마리 토끼를 Generics를 이용해서 구현해 봤습니다.

# 타입 파라미터의 기능 사용하기

드디어 우리가 만든 박스에 기능을 넣어 보도록 하겠습니다. 간단한 연산을 하나 추가할 건데요 박스 데이터와 박스 데이터 2개 평균을 구하는 메소드입니다.

```java
public class NumberGenericBox<T> {
    public double average(NumberGenericBox<T> a){ 
			return ((Double)this.data + (Double)a.getData()) / 2; 
		}
}
```

자기 data를 `double`로 강제 형 변환하고, 인자로 받은 a의 data도 강제로 `double`로 형 변환한 뒤에 둘의 값을 2로 나눈 간단한 평균을 구하는 메소드입니다.

기능이 추가된 Box를 사용해 봅시다.

```java
NumberGenericBox<Double> box1 = new NumberGenericBox<>();
box1.setData(5.0);

NumberGenericBox<Double> box2 = new NumberGenericBox<>();
box2.setData(8.0);

System.out.println(box1.average(box2));
```

결과는 *6.5*로 원하는 답이 나왔습니다. 

그런데 좀 찝찝하지 않으세요?

Box를 사용하는 코드를 이렇게 바꿔보죠.

```java
NumberGenericBox<Integer> box1 = new NumberGenericBox<>();
box1.setData(5);

NumberGenericBox<Integer> box2 = new NumberGenericBox<>();
box2.setData(8);

System.out.println(box1.average(box2));
```

타입 인자를 `Double`에서 `Integer`로 변경했습니다. 실행이 잘 될까요?

잘 안됩니다. `ClassCastException` 이 발생합니다. `average` 메소드를 다시 보면

```java
public double average(NumberGenericBox<T> a){ 
	return ((Double)this.data + (Double)a.getData()) / 2; 
}
```

`this.data` 즉 `Integer` 타입으로 선언된 data를 강제로 `double`로 형 변환하려고 하기 때문입니다.  타입 파라미터로 받은 타입이 `double`로 형 변환을 할 수 있다는 보장이 없습니다. `Integer`는 그래도 되지 않을까? 라는 약간의 기대감은 있지만 `String` 같은 아이는 어림도 없죠. 이렇게 프로그램 안정성이 크게 망가져 버렸습니다. 여기서 고민을 하게 됩니다. 타입 파라미터로 뭔가 불특정 다수의 타입을 받을 수 있도록 하고 싶은데 약간의 제약사항을 걸고 싶은 생각이 듭니다.

이런 고민을 해결해주는 것이 ***경계가 있는 타입 파라미터(Bounded Type Parameter)*** 입니다.

# Bounded Type Parameter

`BoundedGenericBox`에 약간의 제약사항을 넣어 보겠습니다.

```java
public class NumberBoundedGenericBox<T extends Number> {
}
```

`<T extends Number>` 에 주목해주세요. 의미는 간단합니다. 어떤 타입 `T`를 인자로 받을 수는 있는데 `Number` 인터페이스 또는 클래스 이거나 상속받은 타입, 즉 하위 타입만 받을 수 있다는 의미입니다.

이렇게 클래스를 선언하면 Box 클래스를 만들 때 `String` 같은 타입은 사용할 수 없고 `Number` 클래스의 하위 클래스인 `Integer`, `Double` 따위의 클래스만 사용할 수 있습니다. 이렇게 하면 좋은 것은 `Number`가 가지고 있는 기능을 그대로 사용할 수 있다는 것입니다. 

`extends Number`가 없을 때는 `T` 타입의 data가 실제로 `Integer`로 설정이 되더라도 `Integer`가 가지고 있는 기능을 사용할 수 없었습니다. 왜냐하면 컴파일 타임에는 `T`가 어떤 것이 올지 모르니까 가장 최상위 클래스인 `Object` 의 기능만 사용할 수 있도록 하기 때문입니다.

자 그럼 `Number` 클래스가 가지고 있는 기능을 이용해서 `average` 메소드를 다시 정의해 보겠습니다.

```java
public double average(NumberBoundedGenericBox<T> a) {
    return (this.data.doubleValue() + a.getData().doubleValue()) / 2;
}
```

`doubleValue`라는 메소드는 `Number` 클래스에서 제공하는 메소드입니다. 자기 data와 메소드 인자로 받은 box의 값도 `doubleValue()` 메소드를 이용해서 값을 변환한 뒤에 평균을 구하는 코드입니다.

이제 이 박스를 사용해보도록 하겠습니다.

```java
NumberBoundedGenericBox<Integer> b1 = new NumberBoundedGenericBox<>();
b1.setData(5);

NumberBoundedGenericBox<Integer> b2 = new NumberBoundedGenericBox<>();
b2.setData(8);

System.out.println(b1.average(b2));
```

앞선 [리스트 12] 와 같은 코드를 이용해서 `Integer` 박스를 만들고 평균을 구하는 메소드를 실행해 보겠습니다. 실행이 잘 되나요?

잘 됩니다. 이렇게 **타입 파라미터의 범위를 좁혀 줌으로써 타입에 대한 안정성이 더욱 확보**되고 필요한 기능을 세련되게 구현할 수 있었습니다. 타입 인자로 `Ingeter` 대신 `String`을 넣으면 어떻게 될까요?

안됩니다. `Type parameter 'java.lang.String' is not within its bound; should extend 'java.lang.Number'`에러를 뿜으면서 컴파일 타입에 오류를 잡아낼 수 있습니다. 그런데 오류 메시지가 좀 틀린 것 같네요. `Type argument 'java.lang.String'` 이라고 표현 해야 할 것 같은데 말이죠...

여기까지 설명으로 잴 처음 봤던 메소드를 한번 해석해 보겠습니다.

```java
public <T extends Comparable<? super T>> T max(Collection<? extends T> col)
```

음... 일단 위에서 본 것은 클래스 옆에 타입 파라미터를 선언해서 Generic클래스로 만드는 것이었는데 이 코드는 메소드입니다. 아직 안 배웠습니다. 

다만 한가지 알 수 있는 것은 `<T extends Comparable<? super T>>` 에서 불특정의 `T` 타입을 사용할 것인데  `Comparable<? super T>` 이거나 상속한 타입(하위 타입)을 쓴다는 것 정도를 알 수 있습니다. 아직 알아야 할 것이 많네요. 

다음 포스트에서 계속해서 알아가 보겠습니다.

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