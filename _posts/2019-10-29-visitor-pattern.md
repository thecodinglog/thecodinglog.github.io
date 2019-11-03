---
layout: post
title: 방문자 패턴 - Visitor pattern
description: 방문자 패턴 - Visitor pattern
date:   2019-10-29 09:00:00 +0900
author: Jeongjin Kim
categories: Design
tags:	방문자 패턴 VisitorPattern Visitor Design pattern Double Dispatch
---

Visitor pattern, 방문자 패턴, **실제 로직을 가지고 있는 객체(Visitor)**가 로직을 **적용할 객체(Element)**를 **방문**하면서 실행하는 패턴이다.
즉, **로직과 구조를 분리**하는 패턴이라고 볼 수 있다. 로직과 구조가 분리되면 구조를 수정하지 않고도 새로운 동작을 기존 객체 구조에 추가할 수 있다.

이 포스트에서는 주어진 도메인 문제를 해결해가면서 나타나는 문제점을 이야기해보고 이것을 해결해 나가면서 방문자 패턴을 적용해나가는 과정을 설명한다.

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

# 도메인 문제
잘나가 쇼핑몰은 고객을 등급별로 나누고 등급에 따라 차별화된 혜택을 주기로 한다. 고객 등급은 Gold, VIP 가 있고 혜택은 포인트 혜택, 할인 혜택이 있다.
고객 등급별 혜택을 줄 수 있는 확장 가능한 해결책을 찾자.


# 해결 방안
## 고객 등급 객체가 받을 혜택을 직접 구현
등급별로 고객이 받을 혜택이니까 고객 객체에서 직접 받을 수 있는 혜택을 정의하는 방법이다.
먼저, 고객 인터페이스를 사용해서 등급별로 구체 클래스를 만든다.

```java
public interface Member{ }

public class GoldMember implements Member { }

public class VipMember implements Member { }
```
이제 등급별 클래스에 받을 혜택을 구현한다.
```java
public class GoldMember implements Member {
    public void point() { System.out.println("Point for Gold Member"); }
    public void discount() { System.out.println("Discount for Gold Member"); }
}
public class VipMember implements Member {
    public void point() { System.out.println("Point for Vip Member"); }
    public void discount() { System.out.println("Discount for Vip Member"); }
}
```
main 메소드를 만들어서 실행시켜 본다.

```java
public class Main {
    public static void main(String[] args) {
        Member goldMember = new GoldMember();
        Member vipMember = new VipMember();

        goldMember.point();
        vipMember.point();
        goldMember.discount();
        vipMember.discount();
    }
}
```
#### 실행결과
```plain
Point for Gold Member
Point for Vip Member
Discount for Gold Member
Discount for Vip Member
```

고객 등급별로 혜택을 주는 요구사항은 만족한다. 하지만 몇 가지 문제점이 보인다.
1. 고객들을 순회하면서 혜택을 주고자 할 때 명시적으로 혜택을 주기 위한 메소드를 호출해야 한다. 즉, iterator를 이용해서 순회하며 처리할 수 없다.
2. 혜택이 늘어났을 때 모든 멤버들에 대해서 그 혜택을 구현했다는 보장이 없다.

두 가지 문제점을 해결하기 위해서 혜택을 위한 인터페이스를 정의하고 멤버들이 인터페이스를 구현하게 한다.
```java
public interface Benefit {
    void point();
    void discount();
}
public interface Member extends Benefit{ }
```
`Benefit` 인터페이스를 통해 모든 멤버들은 혜택을 구현하도록 하면 `Member` 인터페이스를 통해서 객체들을 순회하며 혜택을 줄 수 있다.

여기서 객체의 역할에 대해서 한번 생각해 볼 필요가 있다. `Member` 객체가 `Benefit`에 대해서 완전히 알고 있어야 할까?
`Benefit`에 대해서 추가하거나 수정한다고 해서 `Member`를 바꿔야 할까? 

만약 고객별로 받을 수 있는 혜택이 추가된다면 모든 `Member` 클래스를 변경해야 해야 한다. 이것은 단일책임 원칙을 완전히 위배하는 행위이다.

그래서 `Member`와 `Benefit` 을 분리해보자.

## 고객과 혜택 클래스를 분리
`Member` Class 에서 혜택과 관련된 코드를 완전히 제거한다.

```java
public interface Member{ }
public class GoldMember implements Member { }
public class VipMember implements Member { }
```
`Benefit` Class 를 구현해서 Member 별 혜택을 구현한다.

```java
public class BenefitImpl implements Benefit {
    @Override
    public void point(Member member) {
        if(member instanceof GoldMember){
            System.out.println("point for Gold Member");
        } else if (member instanceof VipMember) {
            System.out.println("point for Vip Member");
        }
    }

    @Override
    public void discount(Member member) {
        if(member instanceof GoldMember){
            System.out.println("discount for Gold Member");
        } else if (member instanceof VipMember) {
            System.out.println("discount for Vip Member");
        }
    }
}
```
새로 구현한 `Benefit`을 이용해서 실행해 보자.
```java
public class Main {
    public static void main(String[] args) {
        Benefit benefit = new BenefitImpl();

        Member goldMember = new GoldMember();
        Member vipMember = new VipMember();
        benefit.point(goldMember);
        benefit.point(vipMember);
        benefit.discount(goldMember);
        benefit.discount(vipMember);
    }
}
```
처음 구현현한 것과 동일한 결과를 출력한다.
```plain
point for Gold Member
point for Vip Member
discount for Gold Member
discount for Vip Member
```
`BenefitImpl` 클래스는 Member 별로 혜택을 주는 로직을 잘 구현하고 있다. 하지만 눈에 거슬리는 부분이 있다.
```java
if(member instanceof GoldMember)
```
`instanceof` 연산자를 사용해서 Member의 등급을 구별하고 그에 따르는 혜택을 주고 있다. 이 코드 또한 문제가 있는데 

첫 번째, Member 등급이 추가됐을 때 등급별로 혜택을 정확하게 구현했다는 보장이 있는가? 라는 문제이다.

예를 들어 GreenMember가 추가됐는데 그 멤버에 대한 혜택을 주기 위한 코드가 point에 대해서만 했다 하더라도 컴파일러는 개발자에게 어떠한 정보도 주지 않는다. 불완전한 코드를 만들어낼 가능성이 매우 높아진다.

```java
public class BenefitImpl implements Benefit {
    @Override
    public void point(Member member) {
        if(member instanceof GoldMember){
            System.out.println("point for Gold Member");
        } else if (member instanceof VipMember) {
            System.out.println("point for Vip Member");
        }else if(member instanceof GreenMember){
            System.out.println("point for Green Member");
        }
    }

    @Override
    public void discount(Member member) {
        if(member instanceof GoldMember){
            System.out.println("discount for Gold Member");
        } else if (member instanceof VipMember) {
            System.out.println("discount for Vip Member");
        }
    }
}
```

두 번째 문제는 혜택이 추가되면 될수록 같은 구조를 가지는 구분을 반복적으로 사용하게 되어 코드 중복이 일어난다.

freeRent라는 혜택을 모든 멤버에게 주기 위해서는 `instanceof` 를 사용한 분기 구분을 반복해서 사용할 수밖에 없다.

```java
public class BenefitImpl implements Benefit {
    @Override
    public void point(Member member) {
        if(member instanceof GoldMember){
            System.out.println("point for Gold Member");
        } else if (member instanceof VipMember) {
            System.out.println("point for Vip Member");
        }
    }

    @Override
    public void discount(Member member) {
        if(member instanceof GoldMember){
            System.out.println("discount for Gold Member");
        } else if (member instanceof VipMember) {
            System.out.println("discount for Vip Member");
        }
    }

    @Override
    public void freeRent(Member member) {
        if(member instanceof GoldMember){
            System.out.println("freeRent for Gold Member");
        } else if (member instanceof VipMember) {
            System.out.println("freeRent for Vip Member");
        }
    }
}
```

그러면 `Benefit` 인터페이스에 명시적으로 맴버 타입을 받아서 처리하면 되지 않을까?

즉, `Benefit` 인터페이스에 멤버별 메소드를 미리 정의하고 이걸 구현하는 방법이다.

```java
public interface Benefit {
    void point(VipMember member);
    void point(GoldMember member);
    void discount(VipMember member);
    void discount(GoldMember member);
}
```
```java
public class BenefitImpl implements Benefit {
    @Override
    public void point(VipMember member) { System.out.println("point for Vip Member"); }

    @Override
    public void point(GoldMember member) { System.out.println("point for Gold Member"); }

    @Override
    public void discount(VipMember member) { System.out.println("discount for Vip Member"); }

    @Override
    public void discount(GoldMember member) { System.out.println("discount for Gold Member"); }
}
```
이렇게 `Befenfit` 구현하면 `instanceof` 연산자를 사용하지 않고 멤버별 혜택을 줄 수 있다. 이것을 사용하는 클라이언트 코드를 작성해보자.

```java
public class Main {
    public static void main(String[] args) {
        Benefit benefit = new BenefitImpl();
        Member goldMember = new GoldMember();
        Member vipMember = new VipMember();

        benefit.point(goldMember);
        benefit.point(vipMember);
        benefit.discount(goldMember);
        benefit.discount(vipMember);
        benefit.freeRent(vipMember);
        benefit.freeRent(vipMember);
    }
}
```
컴파일되는가? 아쉽게도 이렇게는 사용할 수 없다. 

왜냐하면 `Benefit` 인터페이스의 `point`와 `discount` 메소드는 파라미터 타입으로 구분하여 **오버로딩**을 하고 있다.


>메소드 시그니처 = 메소드 이름 + 파라미터 타입


```java
public void point(VipMember member)
public void point(GoldMember member)
```

두 메소드를 구분하여 실행하기 위해서는 전달받은 파라미터가 `VipMember` 또는 `GoldMember` 이어야 한다.
그런데 `main` 메소드에서 호출한 `point` 메소드는 둘 다 `Member` 타입 파라미터이다.

```java
Member goldMember = new GoldMember();
Member vipMember = new VipMember();

...

benefit.point(goldMember)
benefit.point(vipMember);
```

메소드 오버로딩은 컴파일 타임에 정해진 타입으로 결정하여 연결된다(**Static dispatch**). 따라서 동적으로 변경되는 타입으로 오버로딩으로는 우리가 원하는 결과를 낼 수 없게 된다.

그럼 `Member`를 생성할 때부터 `VipMember` 또는 `GoldMember`로 명시적으로 받으면 되지 않느냐? 라고 할 수 있다.
```java
GoldMember goldMember = new GoldMember();
VipMember vipMember = new VipMember();

...

benefit.point(goldMember)
benefit.point(vipMember);
```
이렇게 하면 물론 컴파일 오류도 나지 않고 현재 이 코드는 실행이 잘 된다. 하지만 이 경우 `Member` 인터페이스를 사용한 이점이 완전히 없어진다. 앞서서 문제점이라고 얘기했던 `Member`들을 순회하며 혜택을 주는 것과 같은 일을 할 수 없다.

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

## Visitor 패턴으로 구현

### 혜택별 Benefit 인터페이스 구현
`Benefit` 인터페이스에 혜택을 받을 `Member` 별로 실행 가능한 메소드를 정의한다.
```java
public interface Benefit {
    void getBenefit(GoldMember member);
    void getBenefit(VipMember member);
}
```
`Benefit` 인터페이스를 구현하는 실제 혜택에 대한 구상 클래스를 구현한다.

```java
public class DiscountBenefit implements Benefit {
    @Override
    public void getBenefit(GoldMember member) {
        System.out.println("Discount for Gold Member");
    }

    @Override
    public void getBenefit(VipMember member) {
        System.out.println("Discount for Vip Member");
    }
}
```
```java
public class PointBenefit implements Benefit {
    @Override
    public void getBenefit(GoldMember member) {
        System.out.println("Point for Gold Member");
    }

    @Override
    public void getBenefit(VipMember member) {
        System.out.println("Point for Vip Member");
    }
}
```
### Member에 혜택을 받을 수 있는 메소드 추가
등급별 멤버가 혜택을 받을 수 있는 메소드를 `Member` 인터페이스에 추가한다.
```java
public interface Member {
    void getBenefit(Benefit benefit);
}
```
`Member` 인터페이스를 구현하는 부분을 보면 인자로 받은 `benefit`의 `getBenefit` 메소드를 호출하는데 이 메소드의 파라미터로 현재 멤버 인스턴스를 넘기도록 한다. 다른 `Member` 가 추가되더라도 구현 부분은 `benefit.getBenefit(this);` 이 한줄만 넣으면 된다.
```java
public class GoldMember implements Member {
    @Override
    public void getBenefit(Benefit benefit) {
        benefit.getBenefit(this);
    }
}
```
```java
public class VipMember implements Member {
    @Override
    public void getBenefit(Benefit benefit) {
        benefit.getBenefit(this);
    }
}
```
이제 이 메소드를 실행할 클라이언트 코드를 작성하고 실행해본다.

```java
public class Main {
    public static void main(String[] args) {
        Member goldMember = new GoldMember();
        Member vipMember = new VipMember();
        Benefit pointBenefit = new PointBenefit();
        Benefit discountBenefit = new DiscountBenefit();

        goldMember.getBenefit(pointBenefit);
        vipMember.getBenefit(pointBenefit);
        goldMember.getBenefit(discountBenefit);
        vipMember.getBenefit(discountBenefit);
    }
}
```
```plain
Point for Gold Member
Point for Vip Member
Discount for Gold Member
Discount for Vip Member
```
원하는 결과가 똑같이 출력됐다. 이전 예제와 같이 Free Rent 혜택을 추가해보자.
혜택을 추가하기 위해서는 Free Rent를 위한 `Benefit` 구상 클래스를 하나 추가하기만 하면 된다.

```java
public class FreeRentBenefit implements Benefit {
    @Override
    public void getBenefit(GoldMember member) {
        System.out.println("FreeRent for Gold Member");
    }

    @Override
    public void getBenefit(VipMember member) {
        System.out.println("FreeRent for Vip Member");
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        ...
        Benefit freeRentBenefit = new FreeRentBenefit();
        ...
        goldMember.getBenefit(freeRentBenefit);
        vipMember.getBenefit(freeRentBenefit);
    }
}
```

### 앞선 문제점을 다 해결하였는가?
앞선 예제에서 발생했던 문제점들을 다시 한번 생각해보자.

`Member` 클래스에 혜택을 모두 구현했을 때는 고객들을 순회하면서 혜택을 주고자 할 때 명시적으로 혜택을 주기 위한 메소드를 호출해야하므로 iterator를 이용해서 순회하며 처리할 수 없었다. 또한 혜택이 늘어났을 때 모든 멤버들에 대해서 그 혜택을 구현했다는 보장이 없다.

Visitor 패턴에서는 `Member` 인터페이스로 등급별 고객을 순회하며 처리할 수 있고, 혜택이 늘어나더라도 `Benefit` 인터페이스에 명시적으로 등급별 메소드를 정의하고 있어 구현 누락이 발생하지 않는다.

`Member` 와 `Benefit` 클래스를 분리하여 구현했을 떄는 Member 등급이 추가됐을 때 등급별로 혜택을 정확하게 구현했다는 보장이 없고 또 혜택이 추가되면 될수록 같은 구조를 가지는 구분을 반복적으로 사용하게 되어 코드 중복이 일어나는 문제가 있었다.

Member를 추가하기 위해 `GreenMember` 구상클래스를 만들고 똑같이 `benefit.getBenefit(this);` 구현 구문을 넣으려고 하면 컴파일 오류가 발생한다. 왜냐하면 `Benefit` 인터페이스에는 `GreenMember` 를 파라미터로 받는 메소드가 존재하지 않기 때문이다. 따라서 등급이 추가되더라도 등급별 혜택 구현을 누락하지 않도록 강제 할 수 있다.

혜택을 추가하기 위해서 한 일은 단지 혜택을 구현하는 클래스를 추가할 뿐이다. 어느 곳에서도 코드 중복을 찾아볼 수 없다.


# Visitor Pattern
지금까지 주어진 도메인 문제를 Visitor Pattern으로 해결해 나가는 과정을 살펴보았다. 이 과정을 토대로 패턴에 대해서 다시 정리해 본다.

## 언제 쓰나?

적용해야 할 **대상 객체가 잘 바뀌지 않고**(특히 개수), 적용할 **알고리즘이 추가될 가능성이 많은 상황**일 때 사용을 고려해봐야 한다. 다시 말하면 Member 등급은 Gold, Vip, Green 으로 고정이거나 추가될 가능성이 작으면서 혜택은 앞으로 계속해서 추가될 가능성이 있을 때를 말한다. 왜냐하면 `Member`가 추가되면 모든 `Benefit` 클래스를 수정해야 하기 때문이다.

또한 대상 객체가 가지는 동작과 객체를 분리해 코드의 응집도를 높이고자 할 때 사용할 수 있다. 멤버별 혜택에 대한 로직을 보기 위해서는 Benefit의 구상 클래스만 보면 쉽게 파악할 수 있다.

## 참여자

이 패턴에 참여자는 **Visitor**, **ConcreteVisitor**, **Element**, **ConcreteElement**, **ObjectStructure** 이다. 책이나 웹사이트에 설명된 것마다 이름만 다를 뿐 나타내고자 하는 것은 같다.

### Visitor
Element를 방문하고 동작을 구현하기 위한 인터페이스이다. (언어마다 인터페이스가 아닐 수 있음)
위 예제에서는 `Benefit` 인터페이스가 그 역할을 한다.
### ConcreteVisitor
실제 알고리즘을 가지고 있는 구현체이다.
위 예제에서는 `PointBenefit`, `DiscountBenefit` 이 그 역할을 한다.
### Element
구조를 구성하는 인터페이스이자 Visitor 가 방문하여 수행해야 할 대상이다. Visitor 를 실행할 수 있는 메소드를 하나 가지고 있으며 보통 `accept` 라는 이름으로 정의한다.
위 예제에서는 `Member` 가 그 역할을 하고 `getBenefit` 메소드가 `accept` 역할을 한다.
### ConcreteElement
**Element**의 실 구현체이다.
위 예제에서는 `VipMember`, `GoldMember` 가 그 역할을 한다.
### ObjectStructure
**Element** 를 가지고 있는 객체 구조이다. 위 예제에서는 특별히 구조를 사용하지 않았다. 보통 `Set`, `List`, `CompositeComponent` 가 그 역할을 한다.


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

# 마무리

도메인에서 사용하는 프로그램을 만들다보면 객체와 객체간 조합 n * n 을 구현해야 할 때가 많았다. 그럴때 마다 `instanceof` 와 같은 연산자로 간단하게 해결해 버리고 말았다. 하지만 시간이 지나다 보니 if 문이 남발되고 유지보수하기가 점점 힘들어져서 해결책을 찾던 도중 **Double dispatch** 기법을 알게되고 이걸 활용한 비지터 패턴을 공부하게 되었다. 다른 패턴에 비해 다소 복잡한것 같기도하고 Double dispatch 기법을 통해 하다보니 한번에 딱 와닫지가 않았는데 처음부터 문제를 해결해 나가는 과정을 보면서 왜 이렇게 써야하는지, 주는 이점이 무엇인지 더 잘 알 수 있게 된 계기가 된것 같다.

이 글을 보는분도 모두 비지터 패턴에 대한 원리를 더 이해를 잘 할 수 있었으면 좋겠다.