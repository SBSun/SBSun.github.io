---
title: "[Spring Data JPA] JPA Projection"
excerpt: "JPA Projection에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring, JPA]

permalink: /spring/spring-jpa-prjection/

toc: true
toc_sticky: true

date: 2023-01-04
last_modified_at: 2023-01-04

--- 

## **Projection**

DB의 필요한 속성만을 조회하는 것을 `projection`이라고 한다.<br>
Spring Data JPA에서 `projection`을 하는 방법을 알아보자.
<br><br>

## **Interface Projection**
<hr/>
조회를 원하는 속성들의 집합으로 인터페이스를 만든다.<br>
원하는 필드들을 `get(필드명)`으로 메서드를 만들어야하고 필드명의 맨 앞은 대문자로 작성한다.

``` java
public interface CommodityInfoProjection {
    int getId();
    String getMarketName();
    String getItemName();
    String getUnit();
    String getPrice();
    String getDate();
}
```
<span style="font-size:130%">**Closed Projection**</span><br>

closed projection은 지정한 필드만을 프로젝션한다.<br>
가져오려는 필드가 무엇인지 먼저 파악이 가능하므로 쿼리 최적화가 가능하다.

위에서 만든 Interface를 Repository Interface에서 사용한다.
``` java
public interface CommodityRepository extends JpaRepository<Commodity, Integer> {
    @Query(value = "SELECT c.commodity_id AS id, m.name AS marketName, i.name AS itemName, c.unit AS unit, c.price AS price, c.p_date AS date " +
        "FROM commodity c " +
        "INNER JOIN market m ON m.gu_id = ?1 AND c.price > 0, item i " +
        "WHERE c.item_id = i.item_id", nativeQuery = true)
    List<CommodityInfoProjection> getCommodityListFromGu(int gu_id);
}
```
Projection Interface로 Repository 메서드를 작성하는 것이 Entity Class와 거의 동일하다는 것을 쉽게 알 수 있다. 유일한 차이점은 Entity Class가 아닌 Projection Interface가 반환된 컬렉션의 요소 유형으로 사용된다는 것이다.

<span style="font-size:130%">**Open Projection**</span><br>

open projection은 일치하지 않는 필드 이름과 런타임에 계산된 반환 값을 사용하여 인터페이스 메서드를 정의할 수 있다.

아래와 같이 `@Value` 어노테이션을 이용해 target으로 지정된 필드를 조합해 새로운 반환 값으로 커스터마이징 할 수 있다.<br>
`@Value` 어노테이션에 들어가는 표현식을 `SpEL`이라고 하는데 SpEL 표현식이 Entity의 모든 필드를 사용할 수 있기 때문에 쿼리 실행 최적화를 적용할 수 없다.
``` java
public interface CommodityInfoProjection {
    @Value("#{target.id + ' ' + target.itemName}")
    String getIdAndItemName();
}
```
<br>

## **Class Projection**
<hr/>
자체 Projection Class를 정의하여 사용할 수 있다.

``` java
@Getter
@AllArgsConstructor
public interface CommodityInfoDto {
    int id();
    String marketName();
    String itemName();
    String unit();
    String price();
    String date();

    // equals and hashCode
}
```
Projection Class가 Repository와 함께 작동하려면 해당 생성자의 매개 변수 이름이 Root Entity Class의 필드와 일치해야 한다.
<br><br>

<span style="font-size:130%">**결론**</span><br>
Interface와 Class 중 선택하여 Projection을 해야겠다.

<hr/>
참고 자료<br>
<a href="https://www.baeldung.com/spring-data-jpa-projections">https://www.baeldung.com/spring-data-jpa-projections</a><br>