---
title: "[Java] 람다식(Lambda Expression)과 함수형 인터페이스(Functional Interface)"
excerpt: "자바의 람다식과 함수형 인터페스에 대해서 알아보자. "

categories:
  - Java
tags:
  - [Java]

published: true

permalink: /java/lambda-functional-interface/

toc: true
toc_sticky: true

date: 2023-02-18
last_modified_at: 2023-02-18

--- 

**Stream** 연산들은 매개변수로 함수형 인터페이스(Functional Interface)를 받도록 되어있다. 그리고 람다식은 반환값으로 함수형 인터페이스를 반환하고 있다.<br>

그렇기 때문에 **Stream API**를 정확히 이해하기 위해 람다식과 함수형 인터페이스에 대해 알아보게 되었다.<br><br>

## **람다식(Lambda Expression)이란?**
<hr />

람다식(Lambda Expression)이란 <span style="color:red">**함수를 하나의 식(expression)으로 표헌한 것**</span>이다.<br>
함수를 람다식으로 표현하면 메서드의 이름이 필요 없기 때문에, 람다식은 <span style="color:red">**익명 함수의 한 종류**</span>라고 볼 수 있다.<br><br>

기존의 함수 선언 방식<br>

``` java
// 기존 방식
반환 타입 메소드명(매개변수, ...){
  실행문
}

// 예시
public String hello(){
  return "Hello World";
}
```
<br>

하지만 람다식은 위와 같이 메서드 명이 불필요하며, 다음과 같이 괄호`()`와 화살표`->`를 이용해 함수를 선언한다.<br>

``` java
// 람다 방식
(매개변수, ...) -> {실행문 ...}

// 예시
() -> "Hello World";
```

이렇게 람다식이 등장하게 된 이유는 <span style="color:red">**불필요한 코드를 줄이고, 가독성을 높이기 위함**</span>이다. 그렇기 때문에 <span style="color:red">**함수형 인터페이스의 인스턴스를 생성하여 함수를 변수처럼 선언**</span>하는 람다식에서는 메서드의 이름이 불필요하다고 여겨져서 이를 사용하지 않는다. 대신 컴파일러가 문맥을 살펴 타입을 추론한다. 또한 람다식으로 선언된 함수는 1급 객체이기 때문에 <span style="color:red">**Stream API의 매개변수로 전달이 가능**</span>해진다.<br><br>

### **람다식의 특징**

**특징**
* 람다식 내에서 사용되는 지역변수는 `final` 키워드가 없어도 상수로 간주된다.
* 람다식으로 선언된 변수명은 다른 변수명과 중복될 수 없다.

**장점**
* 코드를 간결하게 만들 수 있다.
* 식에 개발자의 의도가 명확히 드러나 가독성이 높아진다.
* 함수를 만드는 과정 없이 한 번에 처리할 수 있어 생산성이 높아진다.
* 병렬 프로그래밍이 용이하다.

**단점**
* 람다를 사용하면서 구현한 익명 함수는 재사용이 불가능하다.
* 익명 함수의 특성상 함수 콜 스택 추적이 매우 어려워 디버깅이 어렵다.
* 불필요하게 너무 사용하게 되면 오히려 가독성을 떨어뜨릴 수 있다.

<br>

## **함수형 인터페이스(Functional Interface)란?**
<hr />

* 추상 메서드를 딱 하나만 가지고 있는 인터페이스<br>
  (두 개 이상의 추상 메서드가 있다면 함수형 인터페이스가 아니다.)
* SAM(Single Abstract Method) 인터페이스
* `@FunctionalInterface` 어노테이션을 가지고 있는 인터페이스

기존의 익명 함수는 다음과 같이 작성하였다.<br>

``` java
// 기존의 익명함수
System.out.println(new LambdaFunction() {
    public int max(int a, int b) {
        return a > b ? a : b;
    }
}.max(3, 5));
```
<br>

하지만함수형 인터페이스의 등장으로 <span style="color:red">**함수를 변수처럼 선언**</span>할 수 있게 되었고, 코드 역시 간결하게 작성할 수 있게 되었다.<br> 

함수형 인터페이스를 구현하기 위해서는 인터페이스를 정의하여 그 내부에는 1개 뿐인 abstract 메서드를 선언하고, `@FunctionalInterface` 어노테이션을 추가해주면 된다.<br>

위의 코드를 람다식으로 변경하면 다음과 같다.

``` java
@FunctionalInterface
interface LambdaFunction{
    int max(int a, int b);

    static int sum(int a, int b){
        return a + b;
    }

    default void print(){
        System.out.println("Hello");
    }
}

public class Study {
    public static void main(String[] args) {
        LambdaFunction lambdaFunction = (int a, int b) -> a > b? a : b;
        System.out.println(lambdaFunction.max(3, 5));
        // 5
    }
}
```

위 코드를 보면 `max()` 메서드만 있는 것이 아니라 `sum()`, `print()` 메서드들도 존재하는데 함수형 인터페이스가 아닐까? 답은 **함수형 인터페이스가 맞다** 이다. 중요한건 static 메서드와 default 메서드가 있는게 아니라 **추상 메서드**가 1개만 있어야 한다는 점이다.<br><br>

자바에는 함수형 인터페이스가 이미 정의되어 있다.<br>
다양한 종류가 있지만 해당 포스팅에서는 자주 사용하는 4가지 함수형 인터페이스에 대해서만 알아보자.<br>
* Supplier<T>
* Consumer<T>
* Function<T, R>
* Predicate<T>

<br>

### **Supplier**

* 매개변수 없이 `T` 타입의 반환값 만을 갖는 함수형 인터페이스
* `T get()`을 추상 메서드로 갖는다.

**정의**<br> 

``` java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

`T get()`

``` java
Supplier<String> supplier = () -> "Hello World!";
System.out.println(supplier.get()); // Hello World!
```
<br>

### **Consumer**

* `T` 타입의 매개변수를 받아서 사용하며, 반환값은 없는 항수형 인터페이스<br>
* `void accept(T t)`를 추상 메서드로 갖는다.
* `andThen`이라는 메서드도 제공하는데, 이를 통해 하나의 함수가 끝난 후 다음 **Consumer**를 연쇄적으로 실행할 수 있다.

**정의**<br>

``` java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

`void accept(T t)`

``` java
// 예시
Consumer<String> consumer = (str) -> System.out.println(str.split(" ")[0]);
consumer.andThen(System.out::println).accept("Hello World");

// 출력
Hello
Hello World
```
<br>

### **Function**

* `T` 타입의 매개변수를 받아서 처리한 후 `R` 타입으로 반환하는 함수형 인터페이스
* `R apply(T t)`를 추상 메서드로 갖는다.
* **Consumer**과 마찬가지로 `andThen` 메서드를 제공하고 있다.
* 추가적으로 `compose` 메서드도 제공하는데 `compose`는 첫 번째 함수 실행 이전에 먼저 함수를 실행하여 연쇄적으로 연결해준다는 점에서 `andThen`과 차이점이 존재한다.
* 또한 `identity` 메서드가 존재하는데, 이는 자기 자신을 반환하는 static 메서드이다.

**정의**

``` java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

`R apply(T t)`

``` java
Function<Integer, Integer> plus10 = (i) -> i + 10;
Function<Integer, Integer> multiply2 = (i) -> i * 2;
System.out.println(plus10.andThen(multiply2).apply(2)); // (10 + 2) * 2 = 24
```

`compose`

``` java
Function<Integer, Integer> plus10 = (i) -> i + 10;
Function<Integer, Integer> multiply2 = (i) -> i * 2;
Function<Integer, Integer> multiply2AndPlus10 = plus10.compose(multiply2);
System.out.println(multiply2AndPlus10.apply(2)); // (2 * 2) + 10 = 14
```

`identity`

``` java
final Function<Integer, Integer> identity = Function.identity();
System.out.println(identity.apply(999)); // 999
```

<br>

### **Predicate**

* `T` 타입을 매개변수로 받아 처리한 후 `boolean`을 반환한다.
* `boolean test(T t)`을 추상 메서드로 갖는다.

**정의**

``` java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
``` 

`boolean test(T t)`<br>

``` java
Predicate<String> predicate = str -> str.equals("Hello World");
System.out.println(predicate.test("Hello World")); // true
```

`and`, `or`, `negate`<br>

``` java
Predicate<Integer> isOdd = (i) -> i%2 == 1;
Predicate<Integer> isEven = (i) -> i%2 == 0;

System.out.println(isOdd.and(isEven).test(4)); // fasle, 둘 다 True라면
System.out.println(isOdd.or(isEven).test(4));  // true, 하나라도 True라면
System.out.println(isOdd.negate().test(3));    // true, NOT 연산
```

<hr />
참고자료<br>
<a href="https://mangkyu.tistory.com/113">https://mangkyu.tistory.com/113</a><br>