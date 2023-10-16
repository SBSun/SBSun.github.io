---
title: "[Spring] 빈(Bean)과 스프링 빈을 등록하는 두 가지 방법"
excerpt: " 빈(Bean)과 스프링 빈을 등록하는 두 가지 방법에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring]

published: true

permalink: /spring/bean-and-two-ways-register-spring-bean/

toc: true
toc_sticky: true

date: 2023-03-13
last_modified_at: 2023-03-13

--- 

## **Bean이란?**
<hr />

Bean을 이해하기 위해 스프링 컨테이너에 대해서 알 필요가 있다.<br>

자바 애플리케이션은 애플리케이션 동작을 제공하는 객체들로 이루어져 있다.<br>
이때, 객체들은 독립적으로 동작하는 것 보다 서로 상호작용하여 동작하는 경우가 많다. 이렇게 <span style="color:red">**상호작용하는 객체를 '객체의 의존성'**</span>이라고 표현한다.<br>

스프링에서는 스프링 컨테이너에 객체들을 생성하면 객체끼리 의존성을 주입(DI : Dependency Injection)하는 역할을 해준다. 그리고 <span style="color:red">**스프링 컨테이너에 등록한 객체들을 '빈'**</span>이라고 한다.

<br><br>

## **스프링 컨테이너에 Bean을 등록하는 두 가지 방법**
<hr />

빈을 등록하는 방법은 기본적으로 두 가지 방법이 있다.<br>

**1. 컴포넌트 스캔과 자동 의존관계 설정**<br>
**2. 자바 코드로 직접 스프링 빈 등록**<br>

<br>

### **컴포넌트 스캔과 자동 의존관계 설정**

스프링 부트에서 사용자 클래스를 스프링 빈으로 등록하는 가장 쉬운 방법은 클래스 선언부 위에 `@Component`
어노테이션을 사용하는 것이다.<br>

`@Controller, @Service, @Repository`는 모두 `@Component`를 포함하고 있으며 해당 어노테이션으로
등록된 클래스들은 스프링 컨테이너에 의해 자동으로 생성되어 <span style="color:red">**스프링 빈으로
등록**</span>된다.<br>

``` java
@RestController
public class UserController {}

@Service
public class UserService {}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {}
```

<br>

### **자바 코드로 직접 스프링 빈 등록**

수동으로 스프링 빈을 등록하려면 <span style="color:red">**자바 설정 클래스를 만들어 사용**</span>해야 한다.<br>

설정 클래스를 만들고 `@Configuration` 어노테이션을 클래스 선언부 위에 추가하면 된다.<br>
그리고 특정 타입을 리턴하는 메서드를 만들고, `@Bean` 어노테이션을 붙여주면 자동으로 해당 타입의 빈 객체가 
생성된다.<br>

``` java
@EnableWebSecurity
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception { ... }

    @Bean
    public static BCryptPasswordEncoder bCryptPasswordEncoder() { ... }
}
```

<br>

`@Bean` 어노테이션의 주요 내용은 아래와 같다.<br>

* `@Configuration` 설정된 클래스의 메서드에서 사용 가능
* 메서드의 리턴 객체가 스프링 빈 객체임을 선언
* 빈의 이름은 기본적으로 메서드의 이름
* `@Bean(name="name")`으로 이름 변경 가능
* `@Scope`를 통해 객체 생성을 조정할 수 있다.
* `@Component`를 통해 `@Configuration` 없이도 빈 객체를 생성할 수도 있다.

<br>

어노테이션 하나로 해결되는 1번 방법이 간단하고 많이 사용되고 있지만, 상황에 따라 2번 방법도 사용된다.<br>

1번 방법을 사용해서 개발을 진행하다 UserRepository를 변경해야 할 상황이 생기면 1번 방법은 일일이 변경해줘야 하지만,
 2번 방법을 사용하면 다른건 건들일 필요 없이 @Configuration에 등록된 @Bean만 수정해주면 되므로, 수정이 용이하다.

 <hr />
참고자료<br>
<a href="https://dev-coco.tistory.com/69#recentEntries">https://dev-coco.tistory.com/69#recentEntries</a><br>