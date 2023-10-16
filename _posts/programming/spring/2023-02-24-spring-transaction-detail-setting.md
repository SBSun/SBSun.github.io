---
title: "[Spring] Spring Transaction의 세부 설정"
excerpt: "Spring이 제공하는 Transaction의 세부 설정에 대해서 알아보자."

categories:
  - Spring
tags:
  - [Spring]

published: true

permalink: /spring/transaction-detail-setting/

toc: true
toc_sticky: true

date: 2023-02-24
last_modified_at: 2023-02-24

--- 

<a href="https://sbsun.github.io/spring/transaction-tech/">Spring이 제공하는 Transaction 핵심 기술</a>에 이어서 이번에는 Transaction의 세부 설정에 대해 공부해보자.<br><br>

## **전파 속성(Propagation)**
<hr />

Spring이 제공하는 선언적 트랜잭션(@Transactional)의 장점 중 하나는 여러 트랜잭션 적용 범위를 묶어서 커다란 하나의 트랜잭션 경계를 만들 수 있다는 점이다.<br>

개발자는 Spring이 <span style="color:red">**트랜잭션을 어떻게 진행시킬지 결정하도록 전파 속성을 전달**</span>해야 하는데, 이를 통해 새로운 트랜잭션을 시작할지 또는 기존의 트랜잭션에 참여할지 등을 결정하게 된다.<br>

Spring은 7가지 전파 속성을 지원한다.<br>

* REQUIRED
* SUPPORTS
* MANDATORY
* REQUIRES_NEW
* NOT_SUPPROTED
* NEVER
* NESTED

<br>

**1. REQUIRED**<br>
* 디폴트 속성으로써 모든 트랜잭션 매니저가 지원
* 이미 진행중인 트랜잭션이 있으면 참여하고, 없으면 새로 시작
* 하나의 트랜잭션이 시작된 후, 다른 트랜잭션 경계가 설정된 메서드를 호출하면 같은 트랜잭션으로 묶임

REQUIRED는 기본 속성으로써 모든 트랜잭션 매니저가 지원하며, 대부분의 경우 이 속성이면 충분하다.<br>
이미 진행중인 트랜잭션이 있으면 참여하고 없으면 새로 시작하는 가장 간단하고 자연스러운 트랜잭션 전파 방식이지만 매우 유용하다.<br><br>

**2. SUPPORTS**<br>
* 이미 진행중인 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 진행
* 트랜잭션이 없어도 해당 경계 안에서 Connection 객체나 Hibernate의 Session 등은 공유할 수 있다.

<br>

**3. MANDATORY**<br>              
* 이미 진행중인 트랜잭션이 있으면 참여하고, 없으면 새로 시작하고 예외를 발생시킨다.
* MANDATORY는 혼자서 독립적으로 트랜잭션을 진행하면 안되는 경우에 사용
  
<br>

**4. REQUIRES_NEW**<br>
* 항상 새로운 트랜잭션을 시작해야 하는 경우에 사용
* 이미 진행중인 트랜잭션이 있으면 이를 보류시키고 새로운 트랜잭션을 만들어 시작한다.

<br>

**5. NOT_SUPPORTED**<br>  
* 이미 진행중인 트랜잭션이 있으면 이를 보유시키고, 트랜잭션을 사용하지 않도록 한다.

<br>

**6. NEVER**<br> 
* 이미 진행중인 트랜잭션이 있으면 예외를 발생시키며, 트랜잭션을 사용하지 않도록 강제한다.

<br>

**7.NESTED**<br>
* 이미 진행중인 트랜잭션이 있으면 중첩(자식) 트랜잭션을 시작한다.
* 중첩 트랜잭션은 트랜잭션 안에 다시 트랜잭션을 만드는 것으로, 독립적인 트랜잭션을 만드는 REQUIRES_NEW와는 다르다.
* NESTED에 의한 중첩 트랜잭션은 먼저 시작된 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만, 자신의 커밋과 롤백은 부모 트랜잭션에게 영향을 주지 않는다.
* JDBC 3.0 스펙의 저장포인트(SavePoint)를 지원하는 드라이버와 `DataSourceTransactionManage`를 이용할 경우에 적용할 수 있다.
* `JpaTransactionManager`에서는 지원하지 않는다.

<br><br>

## **격리 수준(Isolation)**
<hr />

트랜잭션의 격리 수준은 <span style="color:red">**동시에 여러 트랜잭션이 진행될 때 트랜잭션의 작업 결과를 여타 트랜잭션에게 어떻게 노출할 것인지를 결정**</span>한다.<br>

Spring은 5가지 격리 수준 속성을 지원한다.<br>

* DEFAULT
* READ_UNCOMMITTED
* READ_COMMITTED
* REPEATABLE_READ
* SERIALIZABLE

<br>

**1. DEFAULT**<br>
* 사용하는 데이터 엑세스 기술 또는 DB 드라이버의 기본 설정을 따른다.
* 일반적으로 드라이버의 격리 수준은 DB의 격리 수준을 따르며, 대부분의 DB는 **READ_COMMITTED**를 기본 격리 수준으로 가진다.  

<br>

**2. READ_UNCOMMITTED**<br>
* 가장 낮은 격리 수준으로써 하나의 트랜잭션이 커밋되지 않은 데이터를 다른 트랜잭션에 그대로 노출되는 문제가 있다.
* 어떤 사용자가 A라는 데이터를 B라는 데이터로 변경하는 동안 다른 사용자는 B라는 아직 완료되지 않은 데이터 B를 읽을 수 있다.
* 하지만 가장 빠르기 때문에 데이터의 일관성이 떨어지더라도 성능을 극대화할 때 의도적으로 사용한다.

<br>

**3. READ_COMMITTED**<br>
* Spring은 기본 속성이 **DEFAULT**이며, DB는 일반적으로 **READ_COMMITTED**가 기본 속성으로 가장 많이 사용된다.
* **READ_UNCOMMITTED**와 달리 다른 트랜잭션이 커밋하지 않은 데이터는 읽을 수 없다.
* 대신 하나의 트랜잭션이 읽은 row를 다른 트랜잭션이 수정할 수 있어서 처음 트랜잭션이 같은 로우를 다시 읽을 때, 다른 내용이 발견될 수 있다.

<br>

**4. REPEATABLE_READ**<br>
* 하나의 트랜잭션이 읽은 row를 다른 트랜잭션이 수정할 수 없도록 막아주지만 새로운 row를 추가하는 것은 막지 않는다.
* 따라서 `SELECT`로 조건에 맞는 row를 전부 가져오는 경우 트랜잭션이 끝나기 전에 추가된 row를 발견할 수 있다.

<br>

**5. SERIALIZABLE**<br>
* 가장 강력한 트랜잭션 격리 수준으로, 완벽한 읽기 일관성 모드를 제공한다.
* 트랜잭션이 완료될 때까지 `SELECT`을 사용하는 모든 데이터에 **shared lock**이 걸리므로 다른 사용자는 그 영역에 해당하는 데이터에 대한 수정 및 입력이 불가능하다.
* 여러 트랜잭션이 동시에 같은 테이블의 정보를 액세스할 수 없다.

<br><br>

## **읽기 전용(readOnly)**
<hr />

Spring에서는 다음의 2가지 목적(<span style="color:red">**성능 최적화와 쓰기 방지**</span>)으로 읽기 전용(readOnly)으로 설정할 수 있다.<br>
* 읽기 전용으로 설정함으로써 성능을 최적화한다.
* 쓰기 작업이 일어나는 것을 의도적으로 방지한다.

<br>

`readOnly`의 기본값은 `false`이며 `true`로 세팅하는 경우 트랜잭션을 읽기 전용으로 변경한다.<br>

만약 읽기 전용 트랜잭션 내에서 INSERT, UPDATE, DELETE 작업을 해도 반영이 되지 않거나 DB 종류에 따라서 예외가 발생하는 경우가 있다.<br><br>

## **롤백/커밋(rollback/commit) 예외**
<hr />

기본적으로 트랜잭션은 종료 시 변경된 데이터를 커밋한다.<br>
하지만 선언적 트랜잭션에서 `rollbackFor` 속성을 지정하면 특정 **Exception** 발생 시 데이터를 커밋하지 않고 롤백하도록 변경할 수 있다.<br>
* 기본값 : RuntimeException, Error
* 사용법 : `@Transactional(rollbackFor = {IOException.class, ClassNotFoundException.class})`

사용할 때 `@Transactional(rollbackFor = IOException.class)` 처럼 Exception을 하나만 지정한다면 중괄호를 생략할 수 있다.<br>

중요한 점은 이 값은 지정된 값이기 때문에 `rollbackFor` 속성으로 다른 Exception을 추가해도 `RuntimeException`이나 `Error`는 여전히 데이터를 롤백한다.<br>

만약 강제로 데이터 롤백을 막고 싶다면 `noRollbackFor` 옵션으로 지정해주면 된다.
<br><br>

## **제한 시간(timeout)**
<hr />

`timeout` 속성을 이용하면 <span style="color:red">**트랜잭션에 제한 시간을 지정**</span>할 수 있다.<br>
값은 int 타입의 초 단위로 지정할 수 있는데 만약 별도로 값을 설정해주지 않는다면 트랜잭션 시스템의 제한 시간을 따른다.<br>
* 기본값 : -1
* 사용법 : `@Transactional(timeout = 2)`

지정한 시간 내에 해당 메서드 수행이 완료되지 않은 경우 `JpaSystemException`을 발생시킨다.<br>
`JpaSystemException`은 `RuntimeException`을 상속받기 때문에 데이터 역시 롤백 처리 된다.<br>

<hr />
참고자료<br>
<a href="https://mangkyu.tistory.com/169">https://mangkyu.tistory.com/169</a><br>
<a href="https://bcp0109.tistory.com/322">https://bcp0109.tistory.com/322</a><br>