---
title: "[Java] 변수(variable)"
excerpt: "자바의 변수에 대해서 알아보자. "

categories:
  - Java
tags:
  - [Java]

permalink: /java/variable/

toc: true
toc_sticky: true

date: 2023-02-15
last_modified_at: 2023-02-15

--- 

## **변수 선언 및 초기화 방법**
<hr />

변수란 데이터를 저장하기 위해 프로그램에 의해 이름을 할당받은 메모리 공간을 의미한다.<br>
즉, 변수란 데이터를 저장할 수 있는 메모리 공간을 의미하며, 이렇게 저장된 값은 변경될 수 있다.<br><br>

### **변수의 선언만 하는 방법**
이 방법은 먼저 변수를 선언하여 메모리 공간을 할당받고, **나중에 변수를 초기화하는 방법**이다.<br>

하지만 이렇게 선언만 된 변수는 초기화되지 않았으므로, 해당 메모리 공간에는 알 수 없는 쓰레깃값만 들어가 있다. 따라서 선언만 된 변수는 반드시 초기화한 후에 사용해야 한다.

``` java
int num;                 // 변수의 선언
System.out.println(num); // 오류 발생
num = 20;                // 변수의 초기화
System.out.println(num); // 20
```
<br>

### **변수의 선언과 동시에 초기화하는 방법**
변수의 선언과 동시에 그 값을 초기화할 수 있다.<br>
또한, 선언하고자 하는 변수들의 타입이 같다면 이를 동시에 선언할 수 있다.

``` java
int num1, num2;                  // 같은 타입의 변수들을 동시에 선언
float num3 = 3.14f;              // 선언과 동시에 초기화
double num4 = 1.23, num5 = 4.56; // 같은 타입의 변수들을 동시에 선언하면서 초기화
```

만약 아래의 코드처럼 같은 타입들의 변수들을 선언한 후 동시에 초기화 하려고 하면, 자바 컴파일러는 오류를 발생시킨다.<br>

``` java
double num1, num2;        // 같은 타입의 변수를 동시에 선언
num1 = 1.23, num2 = 4.56; // 하지만 이미 선언된 여러 변수를 동시에 초기화할 
```
<br>

## **변수에 따른 스코프와 라이프사이클**
<hr />

변수의 스코프란 <span style="color:red">**변수에 접근할 수 있는 유효범위/영역**</span>을 라이프 타임은 <span style="color:red">**변수가 메모리 영역에 저장되어 있는 시간**</span>을 뜻한다.<br>

| <span style="font-size:120%">Variable Type</span> | <span style="font-size:120%">Scope(유효 범위)</span> | <span style="font-size:120%">Life time</span>  | 
|:---:|:---:|:---:|
| <span style="font-size:120%">인스턴스 변수</span> | <span style="font-size:120%">static 메서드를 제외한 클래스 전체</span>  | <span style="font-size:120%">객체가 메모리에 남아있을 때까지</span>  |
| <span style="font-size:120%">클래스 변수</span> | <span style="font-size:120%">클래스 전체</span> | <span style="font-size:120%">프로그램이 끝날때까지 혹은 클래스가 메모리에 로드되는 동안</span> |
| <span style="font-size:120%">지역 변수</span>  | <span style="font-size:120%">변수가 선언된 블록 내부</span>  | <span style="font-size:120%">블록을 벗어날 때까지</span> |

<br>

``` java
public class Study {
    int x, y;          //인스턴스 변수
    static int result; // 클래스 변수

    int add(int a, int b) { // a , b 로컬 변수
        x = a;
        y = b;
        return x + y;
    }
    public static void main(String[] args) {
        Study study = new Study();
        result = study.add(10, 20);
    }
}
```
<br>

## **변수 타입 변환**
<hr />

자바에는 두 종류의 타입 변환이 있다.<br>
* 자동 타입 변환(묵시적)
* 강제 타입 변환(명시적)

<br>

### **자동 타입 변환(promotion)**
프로그램 실행 도중 값의 허용 범위가 작은 타입이 큰 타입으로 저장될 경우 자동 타입 변환이 일어난다.<br>

**타입별 크기 순서(byte)**<br>
`byte(1) < short(2) < int(4) < long(8) < float(4) < double(8)`<br>

<span style="color:red">**float은 표현 범위가 더 크기 때문에 더 큰 타입으로 들어간다.**</span><br>

``` java
byte byNum = 10;
int inNum = byNum;      //자동 타입 변환으로 byNum은 int형으로 변환

double douNum = inNum;  // 10.0 실수 타입으로 변환하면 .0이 붙은 실수값이 됨

char charVal = 'A';
inNum = charVal;        // A의 유니코드 값인 65가 저장
```
<br>

### **강제 타입 변환(casting)**

강제 타입 변환은 큰 크기의 타입을 작은 크기의 타입으로 변환할 때 사용한다.

``` java
int inNum = 'A';                // 65 저장
char charVal = (char)inNum;     // 'A' 저장

double douNum = 3.14;
inNum = (int)douNum;            //정수인 3 저장
```
<br>

## **타입 추론, var**
<hr />

타입 추론이란 타입이 정해지지 않은 변수에 대해서 <span style="color:red">**컴파일러가 변수의 대입된 리터럴로 추론하는 기능**</span>을 말한다.<br>

Java 10에서 `var` 이라는 지역 변수 타입 **추론**이 추가되었다.

* var 타입은 <span style="color:red">**로컬 변수**</span>에만 선언이 가능하다.
* var 타입은 초기화 값을 선언할 때 명시해줘야한다.
* var 타입의 초기화 값은 null이 될 수 없다.

``` java
public class Study {
    var a = 1; // var 타입은 로컬 변수에만 선언이 가능하다.

    public static void main(String[] args) {
        var name = "java";
        var num = 3.14f;
        var list = new ArrayList<String>();

        var i;        // var 타입은 초기화 값을 선언할 때 명시해줘야한다.
        var j = null; // var 타입의 초기화 값은 null이 될 수 없다.
    }
}
```

<hr />
참고자료<br>
<a href="https://dev-coco.tistory.com/2#head10">https://dev-coco.tistory.com/2#head10</a><br>