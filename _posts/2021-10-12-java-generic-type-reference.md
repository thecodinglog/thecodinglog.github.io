---
layout: post
title: 자바 제네릭의 타입 정보를 유지하는 방법 - Super Type Token
description: 자바 제네릭의 타입 정보를 유지하는 방법 - Super Type Token
date:   2021-10-12 18:00:00 +0900
author: Jeongjin Kim
categories: Java
tags:	SuperTypeToken Generics Generic TypeReference
---
> _자바 제네릭의 타입 파라미터를 저장하고 사용하는 방법을 알아봅니다._

제네릭은 **코드의 재사용성**을 높이고 데이터 **타입의 안정성**을 높여줍니다. 제네릭이 없었더라면 객체를 담는 컬렉션을 안전하게 사용할 수 없을 것입니다.

아래는 간단한 컬렉션을 구현하고 사용하는 예제를 살펴봅시다.


```java
public class MyCollection {
    static class SimpleCollection {
        private final Object[] data = new Object[100];
        private int HEAD = 0;

        void add(Object object) {
            data[HEAD] = object;
            HEAD++;
        }

        Object get(int index) {
            return data[index];
        }
    }

    public static void main(String[] args) {
        SimpleCollection collection = new SimpleCollection();
        collection.add("hi");
        collection.add("guys");

        String hi = (String) collection.get(0);
        String guys = (String) collection.get(1);

        System.out.printf("%s %s", hi, guys);
    }
}
```
[코드1] 제네릭을 사용하지 않는 컬렉션과 사용

`SimpleCollection` 에 문자열 두 개를 넣고 출력하는 코드입니다. `SimpleCollection`은 무엇이든 담을 수 있는 컬렉션이기 때문에 사용할 때 매우 유의해야 합니다. 위 코드에서는 컬렉션에서 가져온 값을 `String`으로 형 변환하여 사용하고 있습니다. 실제로 컬렉션에 들어가 있는 값이 `String` 타입이기 때문에 실행을 해도 아무런 오류가 발생하지 않습니다.

값을 가져오는 구문을 아래처럼 변경하면 어떻게 될까요?


```java
Integer hi = (Integer) collection.get(0);
Integer guys = (Integer) collection.get(1);
```
[코드2] 컬렉션의 오브젝트를 강제로 Integer 타입으로 형변환

1. 컴파일 에러가 난다.

2. 런타임 에러가 난다.

3. 에러가 안 난다.


답을 고르셨나요?

<br><br><br>

정답은 2번입니다. 

`SimpleCollection`은 타입에 상관없이 모든 오브젝트를 저장할 수 있는 강력한 기능을 가지고 있지만 저장된 오브젝트를 사용할 때는 타입을 알 수가 없기 때문에 컴파일 에러가 발생하지 않습니다. 이런 특징은 컬렉션을 안전하게 사용하는데 매우 큰 위협이 됩니다.

그러면 타입에 안전하게 사용하기 위해서 컬렉션을 제네릭 타입으로 변경해 보겠습니다.


```java
public class MyCollectionGenerics {
    static class SimpleCollection<T> {
        private final Object[] data = new Object[100];
        private int HEAD = 0;

        void add(T object) {
            data[HEAD] = object;
            HEAD++;
        }

        T get(int index) {
            return (T)data[index];
        }
    }

    public static void main(String[] args) {
        SimpleCollection<String> collection = new SimpleCollection<>();
        collection.add("hi");
        collection.add("guys");

        String hi = collection.get(0);
        String guys = collection.get(1);

        System.out.printf("%s %s", hi, guys);
    }
}
```
[코드3] 제네릭을 사용하는 컬렉션과 사용

`main` 메소드에서 컬렉션을 사용할 때 기존에 있었던 `String` 강제 형 변환 구문이 없어진 것을 알 수 있습니다. 위 코드에서 `String` 타입 변수가 아닌 `Integer` 타입 변수로 아래처럼 할당하게 한다면 어떻게 될까요?

```java
Integer hi = collection.get(0);
Integer guys = collection.get(1);
```
[코드4] 컬렉션의 오브젝트를 강제로 Integer 타입으로 할당


1. 컴파일 에러가 난다.

2. 런타임 에러가 난다.

3. 에러가 안 난다.


답을 고르셨나요?

<br><br><br>
정답은 1번입니다. 

우리는 컴파일 타임에 될 수 있는 한 오류를 찾아내야지 안정적인 프로그램을 만들 수 있습니다. 그런 면에서 제네릭은 매우 훌륭한 도구임이 틀림없습니다.


이번에는 `SimpleCollection<T>` 클래스를 제네릭이 아닌 형태로 사용하면 어떻게 되는지 알아보겠습니다. [코드3]에서 구현한 제네릭 타입 컬렉션과 [코드1]에서 작성한 클라이언트 코드를 단순히 붙여 보겠습니다.

```java
public class MyCollectionGenericsRawType {
    static class SimpleCollection<T> {
        private final Object[] data = new Object[100];
        private int HEAD = 0;

        void add(T object) {
            data[HEAD] = object;
            HEAD++;
        }

        T get(int index) {
            return (T)data[index];
        }
    }

    public static void main(String[] args) {
        SimpleCollection collection = new SimpleCollection();
        collection.add("hi");
        collection.add("guys");

        String hi = (String) collection.get(0);
        String guys = (String) collection.get(1);

        System.out.printf("%s %s", hi, guys);
    }
}
```
[코드5] 제네렉 컬렉션을 일반 컬렉션처럼 사용하기

`SimpleCollection<T>` 은 제네릭 타입 컬렉션이지만 클라이언트 코드인 `main` 메소드에서는 제네릭이 아닌 `Object` 타입으로 그냥 저장했던 방식으로 사용하고 있습니다.

이대로 실행해봅니다.

실행이 잘 되나요?

약간의 warning이 표시되지만, 실행이 잘 될 것입니다. `SimpleCollection<String>` 타입으로 `collection` 을 선언하여 사용해야 하는데 타입 파라미터를 없애버리고 그냥 `SimpleCollection` 으로 `collection` 을 선언하여 사용하고 있습니다. 

`SimpleCollection<String>` 클래스는 자바 1.5에서 작성하여 컴파일된 라이브러리고 클라이언트 코드는 제네릭이 나오기 전인 1.4 이전 버전에서 작성한 코드라고 가정해 봅시다. 만약 이 구문이 실행이 안 된다면 기존에 만들어 놓았던 클라이언트 코드를 모두 바꿔야 하는 상황이 생길 것입니다. 자바는 하위호환성을 매우 매우 중요하게 생각하는 언어입니다. 라이브러리의 버전이 바뀌었다고 클라이언트 코드가 갑자기 실행이 안 되는 현상을 보고만 있을 수는 없을 것입니다. 자바는 이 하위호환성을 유지하기 위해서 컴파일할 때 제네릭 타입을 경계타입이나 `Object`로 바꿔치기해 버립니다.

```java
static class SimpleCollection<T> {
     private final Object[] data = new Object[100];
     private int HEAD = 0;

     void add(T object) {
          data[HEAD] = object;
          HEAD++;
     }

     T get(int index) {
          return (T)data[index];
     }
}
```
[코드6] 제네릭 컬렉션

위 코드에서는 타입 파라미터 `T`를 `Object`로 바꿔치기해서 아래 코드처럼 바뀌게 됩니다.

```java
static class SimpleCollection {
     private final Object[] data = new Object[100];
     private int HEAD = 0;

     void add(Object object) {
          data[HEAD] = object;
          HEAD++;
     }

     Object get(int index) {
          return data[index];
     }
}
```
[코드7] 타입이 변환된 제네릭 컬렉션

바뀐 결과를 보니 제일 처음에 만들었던 컬렉션과 똑같아졌습니다. 그렇기 때문에 [코드5]에서 클라이언트 코드가 정상적으로 동작하게 됩니다. 이렇게 자바는 하위 호환성을 유지하면서 제네릭을 사용할 수 있도록 제네릭 정보를 삭제하여 문제를 해결하고 있습니다.

제네릭 정보를 삭제하는 것은 또 다른 문제를 해결하기 어렵게 만드는데 그 문제를 해결하기 위해 어떤 기법을 사용하는지가 이 포스트의 핵심 내용입니다.


`SimpleCollection<T>` 타입의 오브젝트 두 개를 인자로 받아서 서로 요소 타입이 같으면 병합하는 메소드를 작성한다고 가정해 봅시다. 즉, `["a","b"]` 를 가지고 있는 컬렉션과 `["c","d"]` 컬렉션을 `["a","b","c","d"]` 로 만들어 주는 메소드입니다.

이 문제를 해결하기 위해서는 일단 두 컬렉션이 같은 요소를 가지고 있는 컬렉션인지 확인을 해야 합니다. 같은 요소를 저장할 수 있는 컬렉션인지 확인할 방법이 있을까요?

없습니다. 런타임에는 이미 컴파일러가 제네렉 정보를 지워버렸기 때문입니다.

```java
SimpleCollection<String> stringCollection = new SimpleCollection<>();
SimpleCollection<Integer> integerCollection = new SimpleCollection<>();

System.out.println(stringCollection.getClass().getName());
System.out.println(integerCollection.getClass().getName());
```        
[코드8] 없는 제네릭 정보를 출력해보는 코드

위 코드의 결과는 똑같은 값입니다. 이렇게는 둘 사이를 비교하지 못할 것입니다.

그럼 이번에는 컬렉션을 생성할 때 요소의 타입을 명시적으로 저장하도록 하고 요소 타입을 반환하는 메소드를 추가해봅니다.

```java
public class MyCollectionGenericsType {
    static class SimpleCollection<T> {
        private final Object[] data = new Object[100];
        private int HEAD = 0;
        private final Class<T> elementType;  // 추가

        public SimpleCollection(Class<T> elementType) {  // 추가
            this.elementType = elementType;
        }

        void add(T object) {
            data[HEAD] = object;
            HEAD++;
        }

        T get(int index) {
            return (T) data[index];
        }

        public Class<T> getElementType() {  // 추가
            return elementType;
        }
    }

    public static void main(String[] args) {
        SimpleCollection<String> stringCollection = new SimpleCollection<>(String.class);
        SimpleCollection<Integer> integerCollection = new SimpleCollection<>(Integer.class);

        System.out.println(stringCollection.getElementType().getName());
        System.out.println(integerCollection.getElementType().getName());
    }
}
```
[코드9] 타입을 명시적으로 저장하는 컬렉션

실행해보면 오...

```
java.lang.String
java.lang.Integer
```
요소 정보를 가져올 수 있습니다. 이제 그럼 어떤 요소를 저장하고 있는지 확인한 뒤에 타입이 같으면 하나로 합칠 수 있는 메소드를 만들 수 있을까요?

이 경우는 가능합니다.

그런데 말입니다.

요소 타입이 제네릭 타입이면 어떻게 하면 될까요? `SimpleCollection` 에 `List<String>` 타입 객체를 저장해야 한다면?

일단 한번 해보겠습니다.

```java
SimpleCollection<List<String>> stringCollection = new SimpleCollection<>(List<String>.class);
```

`SimpleCollection<List<String>>` 타입 요소를 저장하는 컬렉션 생성을 시도했지만 할 수 없습니다. `List<String>.class` 에서 오류가 발생했습니다. 왜 그럴까요?

컴파일하고 나면 `List<String>`은 그냥 `List`로 바뀔 것입니다. 그럼 `List.class`로 될 것이고 이 객체를 받는 변수 타입 `SimpleCollection<List<String>>` 와 불일치하므로 오브젝트를 저장할 수 없을 것입니다. 그래서 아래와 같이 RawType 으로 선언하면 컴파일이 되지만 제네릭 타입은 `class` 키워드로 타입 정보를 가져올 수 없습니다.

```java
SimpleCollection<List> listCollection = new SimpleCollection<>(List.class);
```

다시 원점으로 돌아왔습니다.

타입 정보를 가지고 있는 `Class<T>` 의 메소드 중에 `getGenericSuperclass()` 에 주목해 봅니다. 이 메소드는 상위 클래스의 제네릭 타입을 가져오는 메소드입니다. `SimpleCollection<T>`를 상속받는 클래스를 하나 만들어서 인스턴스를 만든 뒤에 `getGenericSuperclass()` 메소드를 호출하여 상위 타입을 가져와 타입 명을 출력해보겠습니다.


```java
import java.lang.reflect.Type;

public class MyCollectionGenericsSuper {
    static class SimpleCollection<T> {
        private final Object[] data = new Object[100];
        private int HEAD = 0;

        void add(T object) {
            data[HEAD] = object;
            HEAD++;
        }

        T get(int index) {
            return (T) data[index];
        }

    }

    static class StringCollection extends  SimpleCollection<String> {   // 추가
    }

    public static void main(String[] args) {
        StringCollection stringCollection = new StringCollection();
        Type genericSuperclass = stringCollection.getClass().getGenericSuperclass();
        System.out.println(genericSuperclass.getTypeName());
    }
}
```
[코드10] 상위 타입을 가져와서 타입을 출력하는 코드

엇!! 제네릭 정보가 손실되지 않고 그대로 출력됩니다.


```
MyCollectionGenericsSuper$SimpleCollection<java.lang.String>
```

더 복잡한 타입을 저장하는 컬렉션으로 바꿔보고 다시 실행해 봅니다.

```java
static class StringCollection extends SimpleCollection<List<Map<String, String>>> {
}
```

정확하게 입력한 그대로 출력되고 있습니다.
```
MyCollectionGenericsSuper$SimpleCollection<java.util.List<java.util.Map<java.lang.String, java.lang.String>>>
```

**자기 자신에게 붙어있는 제네릭정보는 컴파일 타임에 삭제되지만, 부모 클래스가 가지고 있는 제네릭 정보는 그대로 유지하고 있다**는 것을 발견할 수 있습니다. 이 사실을 활용하면 `SimpleCollection` 에 어떤 타입을 저장하고 있는지 정확한 정보를 줄 수 있을 것 같습니다. 간단하게 한번 해보겠습니다.

먼저 항상 부모 클래스가 될 추상 타입 참조 클래스를 만들었습니다. 유효성 검사 같은 것은 모두 생략했으니 유의하세요.

```java
static abstract class TypeReference<T> {
     private final Type type;

     public TypeReference() {
          Type genericSuperclass = getClass().getGenericSuperclass();
          type = ((ParameterizedType) genericSuperclass).getActualTypeArguments()[0];
     }

     public Type getTypeOfSuperClass() {
          return this.type;
     }
}
```

`TypeReference` 추상 클래스는 객체를 만들면서 현재 클래스의 상위타입, 즉 `TypeReference`을 구현한 클래스의 상위타입이니까 `TypeReference` 클래스를 `genericSuperclass` 에 저장합니다. `genericSuperclass` 에서 타입 파라미터에 넘겨받은 타입 인자를 가져와 `type` 필드에 저장합니다. 그리고 `getTypeOfSuperClass` 메소드를 호출하면 최종적으로 타입을 생성할 때 입력한 타입을 반환하게 됩니다.


`SimpleCollection` 클래스를 생성할 때 `Class<T>` 타입이 아닌 `Type`을 받도록 수정합니다.

```java
static class SimpleCollection<T> {
     private final Object[] data = new Object[100];
     private int HEAD = 0;
     private final Type elementType;       // 변경

     public SimpleCollection(TypeReference<T> typeReference) {        // 추가
          this.elementType = typeReference.getTypeOfSuperClass();
     }

     public SimpleCollection(Class<T> tClass) {
          this.elementType = tClass;
     }

     void add(T object) {
          data[HEAD] = object;
          HEAD++;
     }

     T get(int index) {
          return (T) data[index];
     }

     public Type getElementType() {     // 변경
          return elementType;
     }
}
```

이제 `Integer` 타입을 저장하는 컬렉션을 생성해보면 잘 동작함을 알 수 있습니다.

```java
SimpleCollection<Integer> integerCollection = new SimpleCollection<>(Integer.class);
```

사실 `Integer` 타입 컬렉션은 이전에도 됐었죠? 이제 `Integer.class` 자리에 `TypeReference` 를 넣어보도록 합니다. 

```java
SimpleCollection<List<String>> stringCollection = new SimpleCollection<>(
          new TypeReference<List<String>>() {}    //주목
);
```

실행해보면

```
java.util.List<java.lang.String>
java.lang.Integer
```

모두 정확하게 출력되는 것을 확인할 수 있습니다. 소스 코드상에 주목이라고 주석한 부분을 잘 보시기 바랍니다. 일반적으로 객체를 생성하는 방법이 아닌 인라인으로 `TypeReference<T>` 클래스를 구현하고 있습니다.
그렇기 때문에 문장뒤에 `{ } `부분이 존재합니다. 유의해서 사용해야 합니다.


자바는 컴파일 시에 제네릭 정보를 `Object` 타입이나 경계 타입으로 치환을 하므로 제네릭 정보가 삭제됩니다. 하지만 상위 타입의 제네릭 정보는 지워지지 않는다는 점을 이용해서 `TypeReference<T> `같은 추상 클래스를 만들어 정확한 타입을 다룰 수 있습니다.

제네릭도 어려운데 이 개념은 더욱더 어려운 개념입니다. 하지만 타입을 정교하게 다루어야 하는 프로그램을 개발하시는 분은 반드시 숙지해야 하는 중요한 기법의 하나입니다.




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