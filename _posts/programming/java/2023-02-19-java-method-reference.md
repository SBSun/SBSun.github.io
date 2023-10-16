---
title: "[Java] 메소드 참조(Method Reference)"
excerpt: "자바의 메서드 참조에 대해서 알아보자. "

categories:
  - Java
tags:
  - [Java]

published: true

permalink: /java/method-reference/

toc: true
toc_sticky: true

date: 2023-02-19
last_modified_at: 2023-02-19

--- 

## **메서드 참조(Method Reference)란?**
<hr />

메서드 참조란 <span style="color:red">**함수형 인터페이스를 람다식이 아닌 일반 메서드를 참조시켜 선언하는 방법**</span>이다.<br>
일반 메서드를 참조하기 위해서는 다음의 3가지 조건을 만족해야 한다.<br>

* 함수형 인터페이스의 매개변수 타입 = 메서드의 매개변수 타입
* 함수형 인터페이스의 매개변수 개수 = 메서드의 매개변수 개수
* 함수형 인터페이스의 반환 타입 = 메서드의 반환 타입

참조 가능한 메서드는 <span style="color:red">**일반 메서드, static 메서드, 생성자**</span>가 있으며 <span style="color:red">**클래스명::메서드명**</span>으로 참조할 수 있다.<br>
이렇게 참조를 하면 함수형 인터페이스로 반환 된다.<br><br>

### **일반 메서드 참조**

String의 `length()` 메서드는 매개변수가 없으며, 반환형이 int로 동일하기 때문에 `String::length`로 다음과 같이 메서드 참조를 적용할 수 있다.<br>

``` java
// 기존의 람다식
Function<String, Integer> function = str -> str.length();
function.apply("Hello World");

// 메서드 참조로 변경
Function<String, Integer> function = String::length;
function.apply("Hello World");
```
<br>

추가 예시로 `System.out.println()` 메서드는 반환형이 void이며, 매개변수로 String을 받는 메서드이다. 그렇기 때문에 Consumer에 메서드를 참조시킬 수 있다.<br>

``` java
// 일반 메서드를 참조하여 Consumer 선언
Consumer<String> consumer = System.out::println;
consumer.accept("Hello World");

// 메서드 참조를 통해 Consumer를 매개변수로 받는 forEach를 쉽게 사용할 수 있다.
List<String> list = Arrays.asList("Red", "Green", "Blue");
list.forEach(System.out::println);
```
<br>

### **static 메서드 참조**

예를 들어, Objects의 `isNull()` static 메서드는 반환값이 Boolean이며, 매개변수 값은 1개이고 Object 타입이므로 다음과 같이 메서드 참조가 가능하다.<br>

``` java
Predicate<Boolean> predicate = Objects::isNull;
```
<br>

### **생성자 참조**

생성자는 `new` 키워드로 생성해주므로 `클래스이름::new`로 참조할 수 있다.<br>
Supplier는 매개변수가 없이 반환 값만을 갖는 인터페이스이기 때문에, 매개변수 없이 객체를 생성하는 String의 생성자를 참조하여 선언할 수 있다.<br>

``` java
Supplier<String> supplier = String::new;
```

<hr />
참고자료<br>
<a href="https://mangkyu.tistory.com/113">https://mangkyu.tistory.com/113</a><br>