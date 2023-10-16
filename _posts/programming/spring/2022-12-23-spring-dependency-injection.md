---
title: "[Spring] 의존성 주입(Dependency Injection)"
excerpt: "Spring의 의존성 주입(Dependency Injection) 개념과 필요성에 대해 알아보자"

categories:
  - Spring
tags:
  - [Spring]

permalink: /spring/dependency-injection/

toc: true
toc_sticky: true

date: 2022-12-23
last_modified_at: 2022-12-23
--- 

## 의존성 주입(Dependency Injection)의 개념과 필요성
<hr/>

<span style="font-size:120%">**의존성 주입(Dependency Injection) 이란?**</span><br>

의존성 주입(DI)은 <span style="color:red">외부에서 두 객체 간의 관계를 결정해주는 디자인 패턴</span>으로, 인터페이스를 사이에 둬서 클래스 레벨에서는 의존관계가 고정되지 않도록 하고 <span style="color:red">런타임 시에 관계를 동적으로 주입</span>하여 유연성을 확보하고 결합도를 나출 수 있게 해준다.<br>

의존성이란 한 객체가 다른 객체를 사용할 때 의존성이 있다고 한다.<br>
예를 들어, 다음과 같이 Market 객체가 Cake 객체를 사용하고 있는 경우 우리는 Market 객체가 Cake 객체에 의존성이 있다고 표현한다.
``` java
public class Market{
    private Cake cake;
}
```
<br>

<span style="font-size:120%">**의존성 주입이 필요한 이유**</span><br>

예를 들어, Market 클래스와 마켓에서 파는 Cake 클래스가 있다고 하자.
``` java
public class Market{
    private Cake cake;

    public Market(){
        cake = new Cake();
    }
}
```
<br>
위와 같은 예시 클래스는 다음과 같은 문제점을 가지고 있다.<br>

<br>**1. 두 클래스가 강하게 결합되어 있다.**<br>

두 클래스가 강하게 결합되어 있어서 만약 Market에서 Cake가 아닌 Bread와 같은 다른 상품을 판매하고자 한다면 Market 클래스의 생성자에서 변경이 필요하다. 즉, 유연성이 떨어진다. 각각의 다른 상품들을 판매하기 위해 생성자만 다르고 나머지는 중복되는 Market 클래스들이 파생되는 것은 좋지 못하다. 이에 대한 해결책으로 상속을 떠오를 수 있지만, 상속은 제약이 많고 확장성이 떨어지므로 피하는 것이 좋다.<br>

<br>**2. 객체들 간의 관계가 아니라 클래스 간의 관계가 맺어짐**<br>

위의 Market과 Cake는 객체들 간의 관계가 아니라 클래스 간의 관계가 맺어져 있다는 문제가 있다. 올바른 객체지향 설계라면 객체들 간의 관계가 맺어져야 한다. 객체들 간에 관계가 맺어졌다면 다른 객체의 구체 클래스(Cake 인지 Bread 인지 등)를 전혀 알지 못하더라도, 인터페이스 타입으로 사용할 수 있다.<br><br>

결국 이와 같은 문제점이 발생하는 이유는 Market에서 불필요하게 어떤 상품을 판매할 지에 대한 관심이 분리되지 않았기 때문이다. Spring에서는 DI를 적용하여 이와 같은 문제를 해결하고자 하였다.<br><br><br>

##  다양한 의존성 주입 방법
<hr/>

<span style="font-size:120%">**Constructor 주입**</span><br>

생성자를 통해 의존 관계를 주입하는 방법이다.
``` java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

생성자 주입은 생성자의 호출 시점에 1회 호출 되는 것이 보장된다. 그렇기 때문에 주입받은 객체가 변하지 않거나, 반드시 객체의 주입이 필요한 경우에 강제하기 위해 사용할 수 있다. 또한 Spring에서는 생성자 주입을 적극 지원하기 때문에, 생성자가 1개만 있을 경우에 `@Autowired`를 생략해도 주입이 가능하도록 편의성을 제공하고 있다.<br><br>

<span style="font-size:120%">**Setter 주입**</span><br>

필드 값을 변경하는 Setter를 통해서 의존 관계를 주입하는 방법이다.<br>
Setter 주입은 <span style="color:red">주입 받는 객체가 변경될 가능성이 있는 경우</span>에 사용한다.
``` java
@Service
public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
<br>

<span style="font-size:120%">**Field 주입**</span><br>

필드에 바로 의존 관계를 주입하는 방법이다.
``` java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

필드 주입을 이용하면 코드가 간결해져서 과거에 상당히 많이 이용되었던 주입 방법이다. 하지만 필드 주입은 외부에서 접근이 불가능하다는 단점이 존재하는데, 테스트 코드의 중요성이 부각됨에 따라 필드의 객체를 수정할 수 없는 필드 주입은 거의 사용되지 않게 되었다.
<br><br><br>

## 의존성 주입(Dependency Injection)의 개념과 필요성
<hr/>
최근에는 Spring을 포함한 DI 프레임워크의 대부분이 생성자 주입을 권장하고 있는데, 그 이유를 알아보자.
<br><br>

<span style="font-size:120%">**1. 객체의 불변성 확보**</span><br>

Setter 주입이나 일반 메서드 주입을 사용하면 불필요하게 수정의 가능성을 열어두어 유지보수성을 떨어트린다.<br>
그러므로 <span style="color:red">생성자 주입을 통해 변경의 가능성을 배제하고 불변성을 보장</span>하는 것이 좋다.<br><br>

<span style="font-size:120%">**2. 테스트 코드의 작성**</span><br>

테스트가 특정 프레임워크에 의존하는 것은 침투적이므로 좋지 않다. 그러므로 가능한 순수 자바로 테스트를 하는 것이 가장 좋은데, 생성자 주입이 아닌 다른 주입으로 작성된 코드는 순수한 자바 코드로 단위 테스트를 작성하는 것이 어렵다.
``` java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public void save(String id){
        userRepository.save(id);
    }
}
```
<br>

예를 들어 위와 같은 코드를 순수 자바 테스트 코드를 작성하면 다음과 같이 작성할 수 있다.
``` java
public class UserServiceTest{
    @Test
    Public void saveTest(){
        UserService userService = new UserService();
        userService.save("SBS");
    }
}
```

위의 테스트 코드는 Spring 위에서 동작하지 않으므로 의존 관계 주입이 되지 않을 것이고, userRepository가 null이 되어 save 호출 시 NPE가 발생할 것이다. 이를 해결하기 위해 Setter를 사용하면 변경 가능성을 열어두게 되는 단점을 갖게 된다. <br>

반대로 테스트 코드에서 `@Autowired`를 사용하기 위해 Spring을 사용하면 단위 테스트가 아닐 뿐만 아니라, 컴포넌트들을 등록하고 초기화하는 시간 때문에 테스트 비용이 증가하게 된다.<br>

반면에 생성자 주입을 사용하면 <span style="color:red">컴파일 시점에 객체를 주입받아 테스트 코드를 작성</span>할 수 있으며, <span style="color:red">주입하는 객체가 누락된 경우 컴파일 시점에 오류를 발견</span>할 수 있다.<br><br>

<span style="font-size:120%">**3. final 키워드 작성 및 Lombok과의 결합**</span><br>

생성자 주입을 사용하면 필드 객체에 final 키워드를 사용할 수 있으며, <span style="color:red">컴파일 시점에 누락된 의존성을 확인</span>할 수 있다. 반면에 다른 주입 방법들은 객체의 생성(생성자 호출) 이후에 호출되므로 final 키워드를 사용할 수 없다.<br>

또한 final 키워드를 붙이면 <span style="color:red">Lombok과 결합되어 코드를 간결하게 작성</span>할 수 있다. Lombok에는 final 변수를 위한 생성자를 대신 생성해주는 `@RequiredArgsConstructor`가 존재한다. Spring과 같은 DI 프레임워크는 Lombok과 환상적인 궁합을 보여주는데, 위에서 작성했던 생성자 주입 코드를 Lombok과 결합시키면 다음과 같이 간편하게 작성할 수 있다.
``` java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    public void save(String id) {
        userRepository.save(id);
    }
}
```
<br>

<span style="font-size:120%">**4. 스프링에 비침투적인 코드 작성**</span><br>

필드 주입을 사용하려면 `@Autowired`를 이용해야 하는데, 이것은 스프링이 제공하는 어노테이션이다. 그러므로 `@Autowired`를 사용하면 다음과 같이 UserService에 스프링 의존성이 침투하게 된다.
``` java
import org.springframework.beans.factory.annotation.Autowired;
// 스프링 의존성이 UserService에 import되어 코드로 박혀버림

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

우리가 사용하는 프레임워크는 언제 바뀔지도 모를 뿐만 아니라, 사용자와 관련된 책임을 지는 UserService에 스프링 코드가 박혀버리는 것은 바람직하지 않다. 프레임워크는 비즈니스 로직을 작성하는 서비스 계층에서 알아야 할 대상이 아니다.<br><br>

<span style="font-size:120%">**5. 순환 참조 에러 방지**</span><br>

생성자 주입을 사용하면 애플리케이션 구동 시점(객체의 생성 시점)에 순환 참조 에러를 예방할 수 있다.<br>
예를 들어 다음과 같이 필드를 사용해 서로 호출하는 코드가 있다고 하자.
``` java
@Service
public class UserService {
    @Autowired
    private MemberService memberService;
    
    public void save(String id) {
        memberService.save(id);
    }
}
```

UserService가 이미 MemberService에 의존하고 있는데, MemberService 역시 UserService에 의존하는 것이다.
``` java
@Service
public class MemberService {
    @Autowired
    private UserService userService;

    public void save(String id){
        userService.save(id);
    }
}
```

위의 두 메서드는 서로를 계속 호출할 것이고, 메모리에 함수의 CallStack이 계속 쌓여 StackOverflow 에러가 발생하게 된다.<br>

만약 이러한 문제를 발견하지 못하고 서버가 운영된다면 어떻게 되겠는가? 해당 메소드의 호출 시에 StackOverflow 에러에 의해 서버가 죽게 될 것이다. 하지만 생성자 주입을 이용하면 이러한 순환 참조 문제를 방지할 수 있다.<br>

애플리케이션 구동 시점(객체의 생성 시점)에 에러가 발생하기 때문이다. 그러한 이유는 Bean에 등록하기 위해 객체를 생성하는 과정에서 다음과 같이 순환 참조가 발생하기 때문이다.
``` java
new UserService(new MemberService(new UserService(new MemberService()...)))
```

`@Autowired`를 이용한 필드 주입에서 이러한 문제가 애플리케이션 구동 시점에 에러가 발생하지 않는 이유는 <span style="color:red">빈의 생성과 조립(@Autowired) 시점이 분리</span>되어 있기 때문이다. 생성자 주입은 객체의 생성과 조립(의존관계 주입)이 동시에 실행되다 보니 위와 같은 에러를 사전에 잡을 수 있다. 하지만 @Autowired는 모든 객체의 생성이 완료된 후에 조립(의존관계 주입)이 처리된다. 그러다 보니 위와 같이 호출이 되고 나서야 순환 이슈를 확인할 수 있는 것이다.
<br><br><br>

## 의존성 주입(Dependency Injection) 정리
<hr/>
Spring은 의존성 주입을 도와주는 DI 컨테이너로써, <span style="color:red">강하게 결합된 클래스들을 분리</span>하고, 애플리케이션 실행 시점에 <span style="color:red">객체 간의 관계를 결정</span>해 줌으로써 <span style="color:red">결합도를 낮추고 유연성을 확보</span>해준다.

* 두 객체 간의 관계라는 관심사의 분리
* 두 객체 간의 결합도를 낮춘다.
* 객체의 유연성을 높인다.
* 테스트 작성을 용이하게 한다.

<hr/>
참고 자료<br>
<a href="https://mangkyu.tistory.com/150">https://mangkyu.tistory.com/150</a><br>
<a href="https://mangkyu.tistory.com/125">https://mangkyu.tistory.com/125</a><br>