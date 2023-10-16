---
title: "[Spring] @Transactional"
excerpt: "@Transactional 어노테이션에 대해서 알아보자."

categories:
  - Spring
tags:
  - [Spring]

permalink: /spring/transactional/

toc: true
toc_sticky: true

date: 2023-02-12
last_modified_at: 2023-02-12

--- 

## **트랜잭션(Transaction) 이란?**
<hr />

<span style="color:red">**트랜잭션**</span>은 **데이터베이스의 상태를 변경시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위**를 의미한다.<br>

데이터베이스의 상태를 변경시킨다는 것은 무엇일까?<br>

우리는 데이터 하나를 영속하거나 변경하기 위해 여러 작업들을 수행하는데, 이들은 성공할 수도 있고, 실패할 수도 있다.<br>

예를 들어, 내 계좌에 있는 50,000원 중 홍길동이라는 사람에게 30,000원을 송금하려고 한다. 그런데, 네트워크 장애로 인하여 내 계좌의 잔액은 20,000원이 되었지만 홍길동은 30,000원을 받지 못하였다. <br>

이런 경우는 어떻게 문제를 해결할 수 있을까? 

1. 계좌에 있는 50,000원에서 30,000원을 차감한다.
2. 홍길동 계좌에 30,000원을 추가한다.
3. 송금을 완료한다.

이 일련의 과정을 하나의 트랜잭션이라고 하는데, 여기서 정상적으로 송금을 완료하면 <span style="color:red">**Commit(저장)**</span>, 만약 중간에 오류가 발생하면 이 트랜잭션은 실패로 끝나게 되어 내 계좌에 있던 변동 사항은 <span style="color:red">**Rollback(철회)**</span>된다.<br><br>

### **트랜잭션의 성질**

**원자성(Atomicity)**
* 트랜잭션의 연산은 DB에 모두 반영되거나, 전혀 반영되지 않아야 한다.
* 트랜잭션 내의 모든 명령은 반드시 완벽히 수행되어야 하며, 모두가 완벽히 수행되지 않고 어느하나라도 오류가 발생하면 트랜잭션 전부가 취소되어야 한다.
<br>

**일관성(Consistency)**
* 트랜잭션의 작업 처리 결과가 항상 일관되어야 한다.
* 트랜잭션이 진행되는 동안에 DB가 변경 되더라도 변경된 DB로 트랜잭션이 진행된는 것이 아니라, 처음에 트랜잭션을 진행 하기 위해 참조한 DB로 진행되어 각 사용자는 일관성 있는 데이터를 볼 수 있다.

**독립성(Isolation)**
* 둘 이상의 트랜잭션이 동시에 병행 실행되는 경우 어느 하나의 트랜잭션 실행 중에 다른 트랜잭션의 연산이 끼어들 수 없다.
* 수행 중인 트랜잭션이 완료될 때까지, 다른 트랜잭션에서 수행 결과를 참조할 수 없다.

**지속성(Durablility)**
* 트랜잭션이 성공적으로 완료됐을 경우, 결과는 영구적으로 반영되어야 한다.

<br>

## **@Transactional**
<hr />

<span style="color:red">**`@Transactional`**</span>은 Spring Framework에서 제공하는 어노테이션으로 메서드, 클래스, 인터페이스 위에 추가하여 사용하는 방식이 일반적이다. <br>

이 방식을 <span style="color:red">**선언적 트랜잭션**</span>이라 부르며, 적용된 범위에서는 트랜잭션 기능이 포함된 <span style="color:red">**프록시 객체가 생성**</span>되어 자동으로 <span style="color:red">**commit 혹은 rollback을 진행**</span>해준다.<br><br>

### **@Transactional 옵션**

**isolation**
* 트랜잭션에서 일관성없는 데이터 허용 수준을 설정한다.

**propagation**
* 트랜잭션 수행 중 다른 트랜잭션 수행에 끼치는 영향을 설정한다.

**noRollbackFor**
* 특정 예외 발생 시 rollback하지 않는다.

**rollbackFor**
* 특정 예외 발생 시 rollback한다.

**timeout**
* 지정한 시간 내에 메서드 수행이 완료되지 않으면 rollback 한다. (-1인 경우 timeout을 사용하지 않는다, 기본값)

**readOnly**
* 트랜잭션을 읽기 전용으로 사용한다.

<hr />  
참고자료<br>
<a href="https://blog.neonkid.xyz/289">https://blog.neonkid.xyz/289</a><br>