---
title: "[Java] 자바의 자료형(Data Type)"
excerpt: "자바의 자료형에 대해서 알아보자. "

categories:
  - Java
tags:
  - [Java]

permalink: /java/data-type/

toc: true
toc_sticky: true

date: 2023-02-14
last_modified_at: 2023-02-14

--- 

## **자바의 자료형**
<hr />

Java의 자료형에는 기본형(Primitive Type)과 참조형(Reference Type) 2가지가 존재한다.<br>

### **기본형 Primitive Type**

자바에서 기본적으로 제공해주는 기본 자료형으로 primitive type이라 하고 총 8가지의 타입을 미리 정의하여 제공한다.<br>

기본형 데이터 타입은 변수에 <span style="color:red">**값이 직접 저장되어 스택(stack) 영역에 저장**</span>된다.<br>

| 타입 | 메모리 크기 | 기본값  | 표현 범위 |
|---|---|---|---|
| boolean  | 1 byte  | false  | true, false |
| byte     | 1 byte  | 0  | -128 ~ 127 |
| short  | 2 byte  | 0  | -32,768 ~ 32,767 |
| int(기본)  | 4 byte  | 0  | -2,147,483,648 ~ 2,147,483,647  |
| long  | 8 byte  | 0L  | -9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807 |
| float  | 4 byte  | 0.0F | (3.4 X 10-38) ~ (3.4 X 1038) 의 근사값  |
| double(기본)  | 8 byte | 0.0 |  (1.7 X 10-308) ~ (1.7 X 10308) 의 근사값   |
| char  | 2 byte(유니코드)  | '\u0000' | 0 ~ 65,535 |

``` java
public class Study {
    public static void main(String[] args) {
        // 정수형
        byte bNum = 1; // 1byte
        short shortNum = 123; // 2byte
        int num = 1000; // 4byte
        long bigNum = 12345678900L; // long은 뒤에 L을 붙여줘야 한다, 8byte

        // 문자형
        char c = 'a';
        char cNum = 100; // 숫자도 대입할 수 있다.

        // 실수형
        float f = 1.23f; // float는 뒤에 f를 붙여줘야 한다.
        double d = 1.234;

        // 논리형
        boolean isActive = false; // 거짓
        boolean isExists = true; // 참
    }
}
```
<br>

### **참조형 Reference Type** 

프로그래머가 정의한 클래스로 만들어진 자료형으로 reference type이라 한다.<br>

기본형을 제외한 모든 타입을 칭하며 실제 데이터 값이 아닌 <span style="color:red">**객체의 주소값을 저장**</span>한다.<br>
* `Class`, `Interface`, `Array`, `Enumeration` 등
  
참조형 변수는 항상 4 byte의 같은 크기를 가지며 <span style="color:red">**힙(Heap) 메모리에 저장**</span>된다.<br>

선언 시 기본 값으로 Null 값으로 초기화된다.<br>

``` java
public class Study {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3, 4, 5};          // 배열

        String name = "홍길동";                    // 문자열

        List<Integer> scores = new ArrayList<>(); // 리스트
    }
}
```
<br>

## **리터럴과 상수**
<hr />

### **리터럴**

리터럴은 `1, 0.1, 'a'`와 같은 변수에 저장될 수 있는 **변하지 않는 데이터 자체를 의미**한다.

``` java
public class Study {
    public static void main(String[] args) {
        int i = 1;
        String s = "자바";
        double d = 3.14;
        boolean b = true;
    }
} 
```

위와 같이 `=`의 오른쪽 값들이 메모리 위치(공간) 안에 저장되는 리터럴이다.<br><br>

### **상수**

**상수는 메모리 위치(공간)이며, 메모리 값을 변경할 수 없는 불변성을 지니는 변수**이다.<br>

한번 초기화된 상수는 이 후에 다시 초기화 할 수 없으며 자바에서는 `final` 키워드로 상수를 선언한다.

``` java
public class Study {
    public static void main(String[] args) {
        final int i = 1;
        final String s = "자바";
        final double d = 3.14;
        final boolean b = true;
    }
}
```
 
<hr />
참고자료<br>
<a href="https://dev-coco.tistory.com/2">https://dev-coco.tistory.com/2</a><br>

