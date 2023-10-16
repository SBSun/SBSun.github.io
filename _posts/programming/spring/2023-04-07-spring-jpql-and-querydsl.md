---
title: "[Spring Boot] Querydsl에 대해 (1) - JPQL과 Querydsl"
excerpt: "JPQL과 Querydsl에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring Boot, JQPL, Querydsl]

published: true

permalink: /spring/jpql-and-querydsl/

toc: true
toc_sticky: true

date: 2023-04-07
last_modified_at: 2023-04-07

--- 

우아한 테크 유튜브를 보다가 Querydsl에 관한 <a href="https://www.youtube.com/watch?v=zMAX7g6rO_Y&list=WL&index=1&t=165s">영상</a>을 보게 되었다.<br>

해당 영상을 보고 Querydsl에 대해 찾아봤는데 Querydsl의 특징 중 모든 쿼리에 대한 내용을 <span style="color:red">**함수 형태**</span>로 제공되는 점이 JPQL로 쿼리를 작성하고 있던 나의 마음을 돌리기엔 충분했다...<br>

따라서 Querydsl에 대해 자세히 알아보고 구현 및 테스트까지 해보기로 했다.

<br><br>

## **JPQL**
<hr />

JPQL은 Java Persistence Query Language의 약자로 테이블이 아닌 <span style="color:red">**엔티티 객체**</span>를 기준으로 작성하는 **객체 지향 쿼리 언어**라고 정의할 수 있다.<br>

<span style="color:red">**객체**</span>를 기준으로 하기 때문에 개발할 때, Table에 매핑되는 객체가 반드시 존재해야 한다.<br>

**JPQL 특징**<br>
* SQL을 추상화한 JPA의 객체 지향 쿼리
* Table이 아닌 Entity를 대상으로 개발
* JPA는 JPQL을 분석하여 SQL을 생성한 후 DB에서 조회한다.<br>

<br>

**기본 문법**<br>

``` java
String query = "select u from User as u where u.name = 'sbs'";
```

JPQL은 SQL과 문법이 유사하지만 몇 가지 다른 점이 있다.

<br>

**1. 대소문자 구분**<br>
엔티티와 속성은 대소문자를 구분한다.<br>
엔티티 이름이 **User**, 그리고 User의 속성인 **name**은 대소문자를 구분해줘야 한다.<br>
SELECT, FROM, AS 같은 키워드는 구분하지 않아도 된다.

<br>

**2. 엔티티 이름**<br>
위 JPQL 문법에서 사용한 Member는 클래스 이름이 아닌 엔티티 이름이다.<br>
엔티티 이름은  `@Entity(name="abcd")`로 설정 가능하다.<br>
name 속성을 생략하면 기본 값으로 클래스 이름을 사용한다.

<br>

**3. 별칭**<br>
JPQL에서 엔티티의 별칭은 필수적으로 명시해야 한다.<br>
별칭을 명시하는 AS 키워드는 생략할 수 있다.

<br><br>

### **JPQL 구현 방식**

<br>

**1. EntityManager 활용**

EntityManager 객체에서 `createQuery()` 메서드를 호출하면 쿼리가 생성된다.<br>
쿼리 객체로는 <span style="color:red">**TypedQuery**</span>와 <span style="color:red">**Query**</span>가 있는데 반환할 타입을 명확하게 지정할 수 있으면 TypedQuery 객체를, 명확하게 지정할 수 없으면 Query 객체를 사용한다.
<br>

``` java
public static void typedQuery(EntityManager em) {
  String jpql = "select u from User u where u.name = :name";
	TypedQuery<User> query = em.createQuery(jpql, Member.class);
  query.setParameter("name", "SBS");
	
	List<User> list = query.getResultList();
	for( User user : list) {
		System.out.println("User : " + user);
	}
}

public static void Query(EntityManager em) {
  String jpql = "select u from User u where u.name = :name";
	Query query = em.createQuery(jpql);
  query.setParameter("name", "SBS");
	
	List<Object> list = query.getResultList();
	
    for( Object object : list ) {
	      Object[] results = (Object[]) object;
	      
	      for( Object result : results ) {
	          System.out.print ( result );
	     }
	     System.out.println();
	  }
}
```

<br>

**2. Repository Interface 활용**

``` java
@Repository
public interface CommodityRepository extends JpaRepository<Commodity, Integer> {

      @Query(value = "SELECT c1.commodity_id, m.name AS marketName,c1.category_id ,c2.name AS categoryName, " +
            "c1.unit, c1.price, c1.remarks, c1.p_date " +
            "FROM commodity c1 " +
            "INNER JOIN market m ON m.gu_id = ?1 AND c1.market_id = m.market_id " +
            "INNER JOIN category c2 ON c1.category_id = c2.category_id " +
            "WHERE c1.price > 0 " +
            "ORDER BY categoryName " +
            "LIMIT ?2, ?3", nativeQuery = true)
    List<CommodityInfoProjection> getCommodities(int gu_id, int startIndex, int recordSize);
}
```

위 코드는 현재 프로젝트의 구현되어 있는 쿼리문이다.<br>
아직 DB 공부를 자세히 하지 않아서 나중에는 더 깔끔하게 SQL문을 사용할 수 있을 것 같지만 **쿼리를 String 형태로 작성**하고 있어서 오타도 존재할 수 있고 가독성이 좋지 않다고 생각했다.

<br><br>

**JPQL의 문제점**
* JPQL은 **문자열(String)** 형태이기 때문에 **개발자 의존적** 형태이다.
* Compile 단계에서 Type-Check가 불가능하다.
* Runtime 단계에서 오류 발견 가능하다.

<br><br>

## **Querydsl**
<hr />

Querydsl은 **SQL, JPQL 등을 코드로 작성할 수 있도록 해주는 빌더 오픈소스 프레임워크**다.
<br>

기존 방식(Mybatis, JPQL..)은 모두 문자열(String) 형태로 쿼리가 작성되었고 이로 인해 **Compile 단계에서 Type-Check가 불가**했다.<br>
이러한 리스크를 줄이기 위해 Querydsl이 등장했고 이를 통해 **Compile 단계에서 Type-Check가 가능**해졌다.

<br>

### **Querydsl JPA**
Querydsl JPA는 **SQL, JPQL**을 코드로 작성할 수 있도록 해주는 빌더 API이고, Entity 클래스와 매핑되는 <span style="color:red">**QClass**</span> 객체를 사용해서 쿼리를 실행한다.

<br>

### **QClass**
Querydsl은 Compile 단계에서 Entity를 기반으로 QClass를 생성하는데 **JPAAnnotationProcessor**가 컴파일 시점에 작동해서 `@Entity` 등의 어노테이션을 찾아 해당 파일들을 분석해서 QClass를 만든다.
<br>

QClass는 Entity와 형태가 똑같은 Class이다.<br>
Querydsl은 쿼리를 작성할 때 QClass를 기반으로 쿼리를 실행한다.

<br>

### **Querydsl JPA를 사용해야 하는 이유**

1. Querydsl은 쿼리를 **문자열이 아니라 코드를 통해서 작성**하기 때문에 오타가 발생하지 않고, 객체 지향적으로 개발할 수 있다.
2. Querydsl은 코드로 작성하기 때문에 <span style="color:red">**컴파일 단계에서도 오류를 빠르게 발견**</span>할 수 있다.

<br>

**사용 예제**<br>

``` java
public User findByEmail(String email){
    return queryFactory.selectFrom(user)
            .where(user.email.eq(email))
            .fetchOne();
}
```

이런식으로 코드를 작성할 수 있다.<br>
오타가 발생하더라도 컴파일 단계에서 확인할 수 있고, 더욱 객체 지향적으로 개발할 수 있다.

<br>

<hr />
참고자료<br>
<a href="https://velog.io/@cho876/JPQL-vs-query-DSL">https://velog.io/@cho876/JPQL-vs-query-DSL</a><br>

