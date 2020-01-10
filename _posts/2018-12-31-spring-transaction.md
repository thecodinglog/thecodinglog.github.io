---
layout: post
title: Spring Transaction - Multiple DataSources Transaction
description: 다중 데이터 소스를 참조하는 트랜잭션 매니저를 프로그램적으로 관리하기
date:   2018-12-31 11:44:00 +0900
author: Jeongjin Kim
categories: spring
tags:	multi multiple spring transaction 스프링 트랜잭션
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


다중 스프링 트랜잭션을 Programmatic하게 제어하는 방법을 공유하고자 한다. 수작업으로 트랜잭션을 제어하는데 `transaction synchronization is not active` 에러가 뜨면 이 글을 읽어보기 바란다.
## 트랜잭션 경계 설정 방법
스프링에서 트랜잭션을 다루는 방법은 크게 3가지 정도로 나눌 수 있다.
* `@Transactional` 과 같은 Annotation 사용
* AspectJ, Spring AOP 등 수평적 선언 방식
* 프로그램으로 직접 트랜젝션 제어

스프링에서 워낙 쉽고 간결하게 트랜잭션을 관리하는 방법을 제공해 주고 있기 때문에 서비스를 개발하는 프로젝트에서는 개발자가 직접 트랜젝션을 시작하고 종료하는 코드를 작성할 필요가 사실상 없다. 그럼에도 미들웨어나 프레임워크 쪽을 개발하다 보니 수작업으로 관리해야할 일이 있어 간략하게 정리해 본다.

## PlatformTransactionManager - 시작과 종료를 책임
스프링에서 트랜잭션은 `PlatformTransactionManager` 인터페이스가 막중한 책임을 지고있다. 이 인터페이스를 이용해서 트랜잭션을 시작하고 `Commit`, `Rollback` 명령을 수행한다. 이 인터페이스를 구현한 대표적인 클래스가 `DataSourceTransactionManager` 이다. 이 클래스는 DataSource를 필드로 가지고 있고 이 DataSource를 이용해서 `PlatformTransactionManager` 메소드를 구현하고 있다.

이 말은 한 트랜잭션 매니저는 한 DataSource를 가져야하므로 여러 DB에 접근을 해야한다면 TransactionManager도 DataSource 개수에 맞게 준비해야 한다는 의미이다.

## TransactionSynchronizationManager - 트랜잭션을 저장
스프링에서는 Thread 단위로 트랜잭션을 관리할 수 있다. 왜냐하면 `TransactionSynchronizationManager`가 진행중인 트랜잭션 정보와 DataSource 등을 관리하는데 이를 위해서 `ThreadLocal` 에 저장하기 때문이다.

## JpaTransactionManager - 여러 DB 하나의 트랜잭션으로
A DB와 B DB에서 일어났던 작업을 한 트랜잭션으로 묶어서 관리하기 위해서는 `JpaTransactionManager` 를 사용해야한다. 이번 글에서는 다루지 않는다.

## 구현하고 싶은 것
한 서비스에서 여러 DB에 접근해야하고 각 DataSource를 하나의 트랜잭션이 아닌 ___별개의 트랜잭션___ 으로 다루고자 한다. 어떤 경우이냐면 어떤 요청을 받았을 때 이 요청 정보를 서비스의 정상 동작 여부를 떠나 무조건 DB에 로깅하고자 할 때 쓰일 수 있다. 물론 그 로깅하는 부분만 `autoCommit` 설정을 true로 주어 구현할 수도 있지만 등록된 모든 DataSource에서 일관된 정책을 가져야 할 때는 불편할 수 있다.

## 구현방법
인터넷에 검색해보면 프로그램으로 트랜잭션을 제어하는 많은 예제가 있다. 그 중 스프링 [레퍼런스 문서](https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction-programmatic)에 있는 코드 참조하겠다.

```java
@Service
public class TransactionService {
    @Autowired
    private PlatformTransactionManager txManager;

    public void runService() {
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = txManager.getTransaction(def);
        try {
            // execute your business logic here
        } catch (MyException ex) {
            txManager.rollback(status);
            throw ex;
        }
        txManager.commit(status);
    }
}
```

위 코드는 DataSource를 한개 사용하는 일반적인 서비스 수행 코드이다. 만약 `runService` 메소드 안에서 여러 DB에 접속하는 메소드 들이 있을 경우에는 `PlatformTransactionManager`도 접속하는 DataSource 개수만큼 정의를 해야한다.

```java
@Service
public class TransactionService {
    @Autowired
    private PlatformTransactionManager txManager1;
    private PlatformTransactionManager txManager2;
    private PlatformTransactionManager txManager3;

    public void runtService() {
        DefaultTransactionDefinition def1 = new DefaultTransactionDefinition();
        TransactionStatus status1 = txManager1.getTransaction(def1);

        DefaultTransactionDefinition def2 = new DefaultTransactionDefinition();
        TransactionStatus status2 = txManager2.getTransaction(def2);

        DefaultTransactionDefinition def3 = new DefaultTransactionDefinition();
        TransactionStatus status3 = txManager3.getTransaction(def3);
        try {
            // execute your business logic here
        } catch (MyException ex) {
            txManager1.rollback(status1);
            txManager2.rollback(status2);
            txManager3.rollback(status3);
            throw ex;
        }
        txManager1.commit(status1);
        txManager2.commit(status2);
        txManager3.commit(status3);
    }
}
```

Collection을 써서 조금더 단장을 할 수 있지만 간단하게 표현을 했다. 3가지 DataSource와 연결된 `TransactionManager` 들이 있고 try 안에서 뭔가 수행하다가 예외가 발생하면 전부 `rollback`하고 이상없으면 전부 `commit` 하는 코드이다.

하지만 이 코드를 실행해보면 `transaction synchronization is not active` 가 없다고 **예외가 발생**한다.

이를 고치기 위해서는 `rollback`와 `commit`을 하는 `transactionManager`의 순서를 **역순**으로 해야한다.
```java
            txManager3.rollback(status3);
            txManager2.rollback(status2);
            txManager1.rollback(status1);
```            
_이런 식으로_

이유는 간단하다. `getTransaction` 메소드를 호출하면 트랜잭션이 시작되는데 이미 진행중이었던 트랜잭션이 있으면 이걸 suspend 시키고 그 정보를 자신의 `TransactionStatus`에 넣어준다. 새로 시작한 트랜잭션이 `commit`이나 `rollback`이 되면 자기가 suspend 시킨 트랜잭션이 `TransactionStatus`에 있으면 그걸 활성화 시키고 자신을 종료하고, 없으면 전체 트랜잭션은 자체를 비활성화 시켜버린다. 
>첫 번째 시작한 트랜잭션만 가장 나중에 실행되게 해도 지금까지 테스트 해본 결과로는 잘 되었는데 혹시나 하는 마음으로 순서를 꼭 지키는 것이 마음 편한 것 같다.

첫 번째 시작한 트랜잭션은 suspend 정보가 없으니 `rollback`이나 `commit`을 하자마자 전체 트랜잭션을 비활성화하므로 다음 `transactionManager`가 `rollback`, `commit`을 하려고 할 때 `transaction synchronization is not active` 메시지를 띄우면서 예외를 발생시키는 것이다.

이런 방식으로 구현한 트랜잭션 매니저가 있는데 `ChainedTransactionManager` 라는 클래스이다. 구현하려는 목적이 이와 유사하다면 `ChainedTransactionManager` 클래스를 활용해 보는 것도 좋을 것이다.

_유의할 점은 `JpaTransactionManager` 처럼 전체 DataSource를 한 트랜잭션으로 묶는게 아니므로 commit이나 rollback 중에 예외가 발생했을 때 일부만 반영이 된다는 것을 인지하고 있어야 한다._
## 마무리
저 실행 순서를 맞춰야 한다는 사실을 모르고 예외가 발생한 원인을 찾는데 몇 일간 애를 좀 먹었다. 에러가 발생해서 오랜시간 끙끙 찾다보면 정말 별것 아닌것 같은게 원인인 경우가 허다해서 뭔가 성취감이 덜 들때가 많다. 이래서 삽질이라고 하는가 보다.