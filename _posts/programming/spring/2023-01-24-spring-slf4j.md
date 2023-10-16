---
title: "[Spring] @Slf4j란?"
excerpt: "@Slf4j에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring, Slf4j]

permalink: /spring/slf4j/

toc: true
toc_sticky: true

date: 2023-01-24
last_modified_at: 2023-01-24

--- 

## **Slf4j 사용 이유**
<hr/>

프로그램에서 로그를 작성해두면, 어떤 동작을 하고 있는지 혹은 어느 부분에서 에러가 발생하는지 쉽게 파악할 수 있다. 현재까지는 `System.out.print()` 구문을 익숙하게 사용해왔지만 이는 프로그램의 성능을 떨어트리고 로그를 파일에 저장하는 것이 불가능하다.<br>

때문에, Java에서 지원하는 <span style="color:red">**Logging Library**</span>를 사용해서 로그를 관리한다.<br><br>

## **Logging Library 구현체**
<hr/>

<span style="font-size:120%">**log4j2**</span><br>

자바로 개발된 로깅 라이브러리이며 `log4j2`는 `log4j`에서 몇가지 문제점을 개선하여 나온 업그레이드 버전(요즘은 보통 `log4j2`를 사용한다고 한다.)<br>

<span style="font-size:120%">**logback**</span><br>

기존의 `log4j` 이후 같은 개발자가 개발하여 성능이 향상<br>

`Slf4j`의 구현체로 spring boot에서 spring-boot-starter-logging 안에 기본적으로 포험되어 있어서 따로 dependency를 추가하지 않고 사용이 가능한 이점이 있다.<br>

<span style="font-size:120%">**Slf4j**</span><br>

로깅을 간단하게 사용할 수 있도록 하는 Facade<br>

로깅 라이브러리들을 하나의 통일된 방식으로 사용할 수 있는 방법이다.<br><br>

## **@Slf4j란?**
<hr/>

Slf4j는 Simple Logging Facade for Java의 약자로, <span style="color:red">**로깅 추상화 라이브러리**</span>이다.<br>

Logging Framework는 `log4j`, `logback`, `java.util.logging` 등 다양하지만 `@Slf4j` 어노테이션을 통해 `log`객체를 만들어 사용할 수 있으며, 구현체가 변경되더라도 `Slf4j` 덕분에 구현체에 종속되지 않고 사용 가능하다.<br>

Slf4j는 컴파일 시점에 들어있는 의존성 정보로 판단하여 클래스 로더나 메모리 문제가 없어 많이 사용하는 추세이다.<br><br>

## **로그 레벨**
<hr/>

**`TRACE < DEBUG < INFO < WARN < ERROR`**<br>
레벨에 따라서 로그 메세지가 달라진다<br>

* `TRACE` : 추적 레벨은 DEBUG보다 좀 더 상세한 정보를 표시한다.
* `DEBUG` : 프로그램을 디버깅하기 위한 정보를 표시한다.
* `INFO`  : 상태 변경과 같은 정보성 로그를 표시한다.
* `WARN`  : 처리 가능한 문제, 시스템 에러의 원인이 될 수 있는 경고성 메세지를 표시한다.
* `ERROR` : 요청을 처리하는 중 오류가 발생한 경우를 표시한다.
<br><br>

## **Test**
<hr/>

로그를 `+` 문자열 연산자를 통해 파라미터 값을 출력하다 보면 해당 로그가 문자열 연산자를 먼저 실행한 뒤 해당 로그 레벨을 확인하고 출력한다.<br>

이렇게 되면 찍을 필요가 없는 로그에도 문자열 연산자가 실행되어 리소스 낭비가 된다.<br>

여기서 `@Slfj4`의 `{}`를 사용하면 먼저 우선적으로 로그 레벨을 확인한 뒤, 그 뒤로 `{}`에 값을 대입하여 출력한다.<br>

``` java
@Slf4j
@SpringBootTest
public class LogTest {
    @Test
    public void logTest(){
        String value1 = "1번값";
        String value2 = "2번값";
        log.info("1번의 값은 : " + value1 + " 2번의 값은 : "+value2);
        log.info("1번의 값은 : {} 2번의 값은 : {}", value1, value2);
    }
}
```

```
18:07:42.311 [main] INFO backend.jangbogoProject.LogTest - 1번의 값은 : 1번값 2번의 값은 : 2번값
18:07:42.311 [main] INFO backend.jangbogoProject.LogTest - 1번의 값은 : 1번값 2번의 값은 : 2번값
```

<hr/>
참고자료<br>
<a href="https://velog.io/@sjh9391985/Spring-Slf4j%EB%9E%80">https://velog.io/@sjh9391985/Spring-Slf4j%EB%9E%80</a><br>