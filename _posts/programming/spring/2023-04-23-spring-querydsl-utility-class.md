---
title: "[Spring Boot] QueryDSL Utility Class 구현하기"
excerpt: "QueryDSL Utility Class를 구현해보자."

categories:
  - Spring
tags:
  - [Spring Boot, QueryDSL]

published: true

permalink: /spring/querydsl-utility-class/

toc: true
toc_sticky: true

date: 2023-04-23
last_modified_at: 2023-04-23

--- 

QueryDSL로 구현한 Repository 메서드들 중에서 `contains`, `eq`, `ne`, `in` 등의 메서드는 파라미터로 들어가는 반환 타입이나 매개변수 타입만 다를 뿐이지 내부 로직이 똑같은 코드들이 존재한다.<br>

``` java
private BooleanExpression eqId(Long id){
    if(Objects.isNull(id)){
        return null;
    }
    return commodity.id.eq(id);
}

private BooleanExpression eqMarketId(Long marketId){
    if(Objects.isNull(marketId)){
        return null;
    }
    return commodity.M_SEQ.eq(marketId);
}

private BooleanExpression containsName(String name){
    if(!StringUtils.hasText(name)){
        return null;
    }
    return category.name.eq(name);
}
```

그래서 중복 코드를 줄이고 유지 보수를 원할하게 하기 위해 제네릭을 사용해 유틸 클래스를 구현해보자.

<br><br>

## **구현**
<hr />

Q 클래스의 필드들은 기존 필드들의 타입에 따라 `Path` 타입별로 만들어진다.<br>

``` java
public final StringPath A_UNIT = createString("A_UNIT");

public final NumberPath<Long> id = createNumber("id", Long.class);
```
<br>

그래서 `eq` 메서드의 파라미터를 `Path<T>`로 묶어서 작성하면 된다고 생각했지만, `Path<T>` 타입에는 `eq` 메서드가 존재하지 않는다.<br>

``` java
public interface Path<T> extends Expression<T> {
    PathMetadata getMetadata();

    Path<?> getRoot();

    AnnotatedElement getAnnotatedElement();
}
```

애초에 `Path<T>` 타입은 타입과 메타데이터를 가져오기 위한 타입이었고 `BooleanExpression`을 반환하는 메서드들은 추상 클래스쪽에 있었다.<br>

`Expression` 클래스를 타고 올라가보니 `eq` 메서드가 구현된 추상 클래스 `SimpleExpression`를 발견하여 파라미터 타입을 `SimpleExpression<T>` 타입으로 선언했다.
<br>

``` java
public class RepositorySupport {
    
    public static <T> BooleanExpression toEq(final SimpleExpression<T> simpleExpression, final T compared){
        if(Objects.isNull(compared)){
            return null;
        }
        return simpleExpression.eq(compared);
    }

    public static <T> BooleanExpression toEq(final SimpleExpression<T> simpleExpression, final SimpleExpression<T> compared){
        if(Objects.isNull(compared)){
            return null;
        }
        return simpleExpression.eq(compared);
    }

    public static <T> BooleanExpression toNe(final SimpleExpression<T> simpleExpression, final T compared){
        if(Objects.isNull(compared)){
            return null;
        }
        return simpleExpression.ne(compared);
    }

    public static <T> BooleanExpression toNe(final SimpleExpression<T> simpleExpression, final SimpleExpression<T> compared){
        if(Objects.isNull(compared)){
            return null;
        }
        return simpleExpression.ne(compared);
    }

    public static BooleanExpression containsKeyword(final StringPath stringPath, final String keyword) {
        if (!StringUtils.hasText(keyword)) {
            return null;
        }
        return stringPath.contains(keyword);
    }
}
```

현재는 `eq`, `ne`, `contains` 메서드만 구현해서 리팩토링했는데 코드가 훨씬 깔끔해졌다.<br>

``` java
public List<CommodityResponseDto.Info> findByKeyword(Long guId, String keyword, int startIndex, int recordSize) {
        return queryFactory
                .select(Projections.constructor(CommodityResponseDto.Info.class,
                    commodity.id,
                    market.name.as("marketName"),
                    commodity.category_id,
                    category.name.as("categoryName"),
                    commodity.A_UNIT,
                    commodity.A_PRICE,
                    commodity.ADD_COL,
                    commodity.P_DATE
                ))
                .from(commodity)
                .join(market)
                    .on(toEq(market.id, commodity.M_SEQ)
                        , toEq(market.gu_id, guId))
                .join(category)
                    .on(containsKeyword(category.name, keyword))
                .where(toNe(commodity.A_PRICE, "0")
                    , toEq(commodity.category_id, category.id))
                .orderBy(category.name.asc())
                .offset(startIndex)
                .limit(recordSize)
                .fetch();
}
```

앞으로 Repository 기능을 구현하면서 중복 코드들이 발생하면 유틸 클래스에 추가적으로 구현할 예정이다.  
<hr />
참고자료<br>
<a href="https://velog.io/@ohzzi/%EC%9A%B0%EC%95%84%ED%95%9C%ED%85%8C%ED%81%AC%EC%BD%94%EC%8A%A4-4%EA%B8%B0-220810-F12-%EA%B0%9C%EB%B0%9C%EC%9D%BC%EC%A7%80">https://velog.io/@ohzzi/</a><br>