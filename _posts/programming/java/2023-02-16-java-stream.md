---
title: "[Java] 스트림(Stream)"
excerpt: "자바의 스트림(Stream)에 대해서 알아보자. "

categories:
  - Java
tags:
  - [Java]

published: true

permalink: /java/stream/

toc: true
toc_sticky: true

date: 2023-02-16
last_modified_at: 2023-02-16

--- 

## **스트림(Stream)이란?**
<hr />

스트림(Stream)은 '데이터의 흐름'이다. 자바 8부터 추가된 <span style="color:red">**컬렉션(배열 포함)의 저장 요소를 하나씩 참조해서 람다식으로 처리**</span>할 수 있도록 해주는 반복자이다.<br>

자바 8 이전에는 배열 또는 컬랙션 인스턴스를 다루는 방법은 `for` 또는 `foreach` 문을 돌면서 요소 하나씩을 꺼내 다루는 방법이었다. 간단한 경우라면 상관없지만 로직이 복잡해질수록 코드의 양이 많아져 여러 로직이 섞이게 되고, 메서드를 나눌 경우 루프를 여러 번 도는 경우가 발생한다.<br>

Iterator와 비슷한 역할을 하지만 람다식으로 요소 처리 코드를 제공하여 <span style="color:red">**코드가 좀 더 간결**</span>하게 할 수 있는 점과 내부 반복자를 사용하므로 <span style="color:red">**병렬처리가 쉽다**</span>는 점에서 차이가 있다.<br>

또 하나의 장점은 배열 또는 컬랙션 인스턴스에 함수 여러 개를 조합해서 <span style="color:red">**원하는 결과를 필터링하고 가공된 결과를 얻을 수 있다.**</span><br>

스트림에 대한 내용은 크게 세 가지로 나눌 수 있다.<br>
1. 생성하기 : 스트림 인스턴스 생성
2. 가공하기 : 필터링(filtering) 및 맵핑(mapping) 등 원하는 결과를 만들어가는 중간 작업
3. 결과 만들기 : 최종적으로 결과를 만들어내는 작업

이제부터 각 단계에 대해 알아보고 실습해보자.<br><br>

## **생성하기**
<hr />

보통 배열과 컬렉션을 이용해서 스트림을 만들지만 이 외에도 다양한 방법으로 스트림을 만들 수 있다.<br>

스트림을 이용하기 위해서는 먼저 생성을 해야 한다. 스트림은 배열 또는 컬렉션 인스턴스를 사용해서 생성할 수 있다.

### **배열 스트림**
배열은 다음과 같이 `Arrays.stream()` 메서드를 사용한다.<br>

``` java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream1 = Arrays.stream(arr);
Stream<String> stream2 = Arrays.stream(arr, 1, 3);  // arr 배열의 1~2 요소 [b, c]
```
<br>

### **컬렉션 스트림**
컬렉션 타입(List, Set)의 경우 인터페이스에 추가된 디폴트 메서드 `stream`을 이용해서 스트림을 만들 수 있다.<br>

``` java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```
<br>

### **비어 있는 스트림**
비어 있는 스트림(empty stream)도 생성할 수 있다. 빈 스트림은 요소가 없을 때 `null` 대신 사용할 수 있다.<br>

``` java
public Stream<String> streamOf(List<String> list) {
    return list == null || list.isEmpty()
            ? Stream.empty() : list.stream();
}
```
<br>

### **Stream.builder()**
빌더(builder)를 사용하면 스트림에 직접적으로 원하는 값을 넣을 수 있다. 마지막에 `build()` 메서드로 스트림을 리턴한다.<br>

``` java
Stream<String> stream =
        Stream.<String>builder()
                .add("a").add("b").add("c")
                .build();
```
<br>

### **Stream.generate()**
`generate()` 메서드를 이용하면 `Supplier<T>`에 해당하는 람다로 값을 넣을 수 있다. `Supplier<T>`는 인자는 없고 리턴 값만 있는 함수형 인터페이스다.<br>

``` java
public static<T> Stream<T> generate(Supplier<T> s) { ... }
```

이때 생성되는 스트림은 크기가 정해져있지 않고 무한하기 때문에 특정 사이즈로 최대 크기를 제한해야 한다.<br>

``` java
Stream<String> stream =
        Stream.generate(() -> "str").limit(5); // [str, str, str, str, str]
```

5개의 "str" 이 들어간 스트림이 생성된다.<br><br>

### **Stream.iterate()**
`iterate()` 메서드를 이용하면 초기 값과 해당 값을 다루는 람다를 이용해서 스트림에 들어갈 요소를 만든다. <br>

다음 예제에서는 30이 초기값이고 값이 2씩 증가하는 값들이 들어가게 된다.<br>
즉, 요소가 다음 요소의 인풋으로 들어간다. 이 방법도 스트림의 사이즈가 무한이기 때문에 특정 사이즈로 제한해야 한다.<br>

``` java
Stream<Integer> stream =
        Stream.iterate(30, num -> num + 2).limit(5); // [30, 32, 34, 36, 38]
```

<br>

## **가공하기**
<hr />

전체 요소 중에서 다음과 같은 API를 이용해서 내가 원하는 것만 뽑아낼 수 있다. 이러한 가공 단계를 중간 작업이라고 하는데, 이러한 작업은 스트림을 리턴하기 때문에, 여러 작업을 이어 붙여서 작성할 수 있다.<br>

``` java
List<String> names = Arrays.asList("Eric", "Elena", "Java");
```

아래 나오는 예제 코드는 위와 같은 리스트를 대상으로 한다.<br>

### **Filtering**
필터(filter)는 스트림 내 요소들을 하나씩 평가해서 걸러내는 작업이다. 인자로 받는 `Predicate`는 boolean을 리턴하는 함수형 인터페이스로 평가식이 들어가게 된다.<br>

``` java
Stream<T> filter(Predicate<? super T> predicate);
```

``` java
Stream<String> stream =
        names.stream()
                .filter(name -> name.contains("a")); // [Elena, Java]
```
스트림의 각 요소에 대해서 평가식을 실행하게 되고 `a`가 포함된 이름만을 가진 스트림이 리턴된다.<br><br>

### **Mapping**
맵(map)은 스트림 내 요소들을 하나씩 특정 값으로 변환해준다. 이 때 값을 변환하기 위한 람다를 인자로 받는다.<br>

``` java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

스트림에 들아가 있는 값이 input 되어서 특정 로직을 거친 후 output이 되어(리턴되는) 새로운 스트림에 담기게 된다. 이러한 작업을 맵핑(mapping)이라고 한다.<br>

``` java
Stream<String> stream =
        names.stream()
                .map(String::toUpperCase); // [ERIC, ELENA, JAVA]
```

스트림 내 String의 `toUpperCase()` 메서드를 실행해서 대문자로 변환한 값들이 담긴 스트림을 리턴한다.<br>

`map` 이외에도 조금 더 복잡한 `findmap` 메서드도 있다.<br>

``` java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)
```

인자로 `mapper`를 받고 있는데, 리턴 타입이 Stream이다. 즉, 새로운 스트림을 생성해서 리턴하는 람다를 넘겨야한다. `flatMap`은 중첩 구조를 한 단계 제거하고 단일 컬렉션으로 만들어주는 역할을 한다. 이러한 작업을 플래트닝(flattening)이라고 한다.<br>

다음과 같은 중첩된 리스트가 있다.<br>

``` java
List<List<String>> list = 
  Arrays.asList(Arrays.asList("a"), 
                Arrays.asList("b"));
// [[a], [b]]
```

이를 `flatMap`을 사용해서 중첩 구조를 제거한 후 작업할 수 있다.<br>

``` java
List<String> flatList =
                list.stream()
                        .flatMap(Collection::stream)
                        .collect(Collectors.toList());
// [a, b]
```
<br>

### **Sorting**
정렬의 방법은 다른 정렬과 마찬가지로 Comparator를 이용한다.

``` java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```

인자 없이 그냥 호출할 경우 오름차순으로 정렬한다.<br>

``` java
List<Integer> list =
        IntStream.of(14, 11, 20, 28, 23)
                .sorted()
                .boxed()
                .collect(Collectors.toList());
// [11, 14, 20, 23, 28]
```

인자를 넘기는 경우와 비교해보자. String List에서 알파벳 순으로 정렬한 코드와 Comparator를 넘겨서 역순으로 정렬한 코드이다.<br>

``` java
List<String> lang = 
  Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");

lang.stream()
  .sorted()
  .collect(Collectors.toList());
// [Go, Groovy, Java, Python, Scala, Swift]

lang.stream()
  .sorted(Comparator.reverseOrder())
  .collect(Collectors.toList());
// [Swift, Scala, Python, Java, Groovy, Go]
```

기본적으로 Comparator 사용법과 동일하다. 이를 이용해서 문자열 길이를 기준으로 정렬해보자.<br>

``` java
lang.stream()
  .sorted(Comparator.comparingInt(String::length))
  .collect(Collectors.toList());
// [Go, Java, Scala, Swift, Groovy, Python]

lang.stream()
  .sorted((s1, s2) -> s2.length() - s1.length())
  .collect(Collectors.toList());
// [Groovy, Python, Scala, Swift, Java, Go
```
<br>

### **Iterating**
스트림 내 요소들 각각을 대상으로 특정 연산을 수행하는 메서드로는 `peek()`이 있다.<br>
`peek`은 그냥 확인해본다는 단어 뜻처럼 특정 결과를 반환하지 않는 함수형 인터페이스 Consumer를 인자로 받는다.<br>

``` java
Stream<T> peek(Consumer<? super T> action);
```

따라서 스트림 내 요소들 각각에 특정 작업을 수행할 뿐 결과에 영향을 미치지 않는다.<br>
다음처럼 작업을 처리하는 중간에 결과를 확인해볼 때 사용할 수 있다.<br>

``` java
int sum = IntStream.of(1, 3, 5, 7, 9)
        .peek(System.out::println)
        .sum();
```

<br>

## **결과 만들기**
<hr />

가공한 스트림을 내가 사용할 결과값으로 만들어내는 단계로 스트림을 끝내는 최종 작업이다.<br>

### **Calculating**
스트림 API는 다양한 종료 작업을 제공한다. 최소, 최대, 합, 평균 등 기본형 타입으로 결과를 만들어낼 수 있다.<br>

``` java
long cnt = LongStream.of(1, 3, 5, 7, 9).count();
int sum = IntStream.of(1, 3, 5, 7, 9).sum();
```

만약 스트림이 비어 있는 경우 `count`와 `sum`은 0을 출력하면 된다. 하지만 평균, 최소, 최대의 경우에는 표현할 수가 없기 때문에에 Optional을 이용해 리턴한다.<br>

``` java
DoubleStream.of(1.1, 2.2, 3.3, 4.4, 5.5)
        .average()
        .ifPresent(System.out::println);

OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max();
```
<br>

### **Reduction**
스트림은 `reduce()`라는 메서드를 이용해서 결과를 만들어낸다.<br>

`reduce()` 메서드는 총 세 가지의 파라미터를 받을 수 있다.

* accumulator : 각 요소를 처리하는 계산 로직. 각 요소가 올 때마다 중간 결과를 생성하는 로직.
* identity : 계산을 위한 초기값으로 스트림이 비어서 계산할 내용이 없더라도 이 값은 리턴된다.
* combiner : 병렬 스트림에서 나눠 계산한 결과를 하나로 합치는 로직.

``` java
// 1개 (accumulator)
Optional<T> reduce(BinaryOperator<T> accumulator);

// 2개 (identity)
T reduce(T identity, BinaryOperator<T> accumulator);

// 3개 (combiner)
<U> U reduce(U identity,
  BiFunction<U, ? super T, U> accumulator,
  BinaryOperator<U> combiner);
```

먼저 인자가 하나만 있는 경우이다. 여기서 `BinaryOperator<T>`는 같은 타입의 인자 두 개를 받아 같은 타입의 결과를 반환하는 함수형 인터페이스이다.<br>
다음 코드에서는 두 값을 더하는 람다를 넘겨주고 있다. 따라서 결과는 6(1 + 2 + 3)이 된다.<br>

``` java
OptionalInt reduced =
        IntStream.range(1, 4) // [1, 2, 3]
                .reduce((a, b) -> {
                    return Integer.sum(a, b);
                });
```

이번엔 두 개의 인자를 받는 경우이다. 여기서 10은 초기값이고, 스트림 내 값을 더해서 결과는 16이 된다. 여기서 람다는 <span style="color:red">**메서드 참조**</span>를 이용해서 넘길 수 있다.<br>

``` java
int reduced =
        IntStream.range(1, 4) // [1, 2, 3]
                .reduce(10, Integer::sum); // method reference
```

마지막으로 세 개의 인자를 받는 경우이다. Combiner는 병렬 처리 시 각자 다른 쓰레드에서 실행한 결과를 마지막에 합치는 단게이다. 따라서 병렬 스트림에서만 동작한다.<br>

``` java
Integer reducedParallel = Arrays.asList(1, 2, 3)
  .parallelStream()
  .reduce(10, // identity
          Integer::sum, // accumulator
          (a, b) -> {
            System.out.println("combiner was called");
            return a + b;
          }); 
```

결과는 36이 나온다. 먼저 accumulator는 총 세 번 동작한다. 초기값 10에 각 스트림 값을 더한 세 개의 값(`10 + 1 = 1, 10 + 2 = 12, 10 + 3 = 13`)을 계산한다.<br>

Combiner는 identity와 accumulator를 가지고 여러 쓰레드에서 나눠 계산한 결과를 합치는 역할이다. <br>
`12 + 13 = 25`, `25 + 11 = 36` 이렇게 두 번 호출된다.<br><br>

### **Collecting**
`collect()` 메서드는 `Collector` 타입의 인자를 받아 처리한다. 자주 사용하는 작업은 `Collectors` 객체에서 제공한다.<br>

이번 예제는 다음과 같은 리스트를 사용한다. Product 객체는 수량(amount)과 이름(name)을 가지고 있다.<br>

``` java
List<Product> productList = 
  Arrays.asList(new Product(23, "potatoes"),
          new Product(14, "orange"),
          new Product(13, "lemon"),
          new Product(23, "bread"),
          new Product(13, "sugar"));
```
<br>

**Collectors.toList()**<br>

스트림에서 작업한 결과를 다음 리스트로 반환한다.<br>
다음 예제에서는 `map()`으로 각 요소의 이름을 가져온 후 `Collectors.toList()`를 이용해서 리스트로 결과를 가져온다.<br>

``` java
List<String> names =
        productList.stream()
                .map(Product::getName)
                .collect(Collectors.toList()); 
// [potatoes, orange, lemon, bread, sugar]
```
<br>

**Collectors.joining()**<br>

스트림에서 작업한 결과를 하나의 String으로 이어 붙인 결과를 리턴한다.<br>

``` java
String listToString =
        productList.stream()
                .map(Product::getName)
                .collect(Collectors.joining());
// potatoesorangelemonbreadsugar
```

`Collectors.joining()`은 세 개의 인자를 받을 수 있다.<br>
* delimiter : 각 요소 중간에 들어가 요소를 구분시켜주는 구분자
* prefix : 결과 맨 앞에 붙는 문자
* suffix : 결과 맨 뒤에 붙는 문자

``` java
String listToString =
        productList.stream()
                .map(Product::getName)
                .collect(Collectors.joining(",", "<", ">"));
// <potatoes, orange, lemon, bread, sugar>
```
<br>

**Collectors.averageingInt()**<br>

숫자 값(Integer value)의 평균을 반환한다.<br>

``` java
Double averageAmount =
        productList.stream()
                .collect(Collectors.averagingInt(Product::getAmount));
// 17.2
```
<br>

**Collectors.summingInt()**<br>

숫자 값의 합을 반환한다.<br>

``` java
Integer sumAmount = 
        productList.stream()
                        .collect(Collectors.summingInt(Product::getAmount));
// 86
```
<br>

### **Matching**
매칭은 조건식 람다 `Predicate`를 받아서 해당 조건을 만족하는 요소가 있는지 체크한 결과를 리턴한다.<br>
* anyMatch : 하나라도 조건을 만족한 요소가 있는지
* allMatch : 모두 조건을 만족하는지
* noneMatch : 모두 조건을 만족하지 않는지

``` java
boolean anyMatch(Predicate<? super T> predicate);
boolean allMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);
```

다음 코드의 결과는 모두 `true`이다.<br>

``` java
List<String> names = Arrays.asList("Eric", "Elena", "Java");

boolean anyMatch = names.stream()
        .anyMatch(name -> name.contains("a"));
boolean allMatch = names.stream()
        .allMatch(name -> name.length() > 3);
boolean noneMatch = names.stream()
        .noneMatch(name -> name.endsWith("s"));
```
<br>

### **Iterating**
`foreach`는 요소를 돌면서 실행되는 최종 작업이다.<br>
보통 `System.out.println()` 메서드를 넘겨서 결과를 출력할 때 사용하곤 한다.<br>

``` java
names.stream().forEach(System.out::println);
```

<hr />
참고자료<br>
<a href="https://futurecreator.github.io/2018/08/26/java-8-streams/">https://futurecreator.github.io/2018/08/26/java-8-streams/</a><br>