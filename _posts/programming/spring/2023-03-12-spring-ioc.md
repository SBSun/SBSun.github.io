---
title: "[Spring] IoC(Inversion of Control)란?"
excerpt: "IoC에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring]

published: true

permalink: /spring/ioc/

toc: true
toc_sticky: true

date: 2023-03-12
last_modified_at: 2023-03-12

--- 

## **IoC(Inversion of Control)**
<hr />

IoC(제어 반전)이란, 객체의 생성, 생명 주기의 관리까지 모든 객체에 대한 제어권이 바뀌었다는 것을 의미한다.<br>

컴포넌트 의존 관계 설정(Component dependency resoulution), 설정(Configuration) 및 생명 주기(LifeCycle)를 해결하기 위한 디자인 패턴이다.<br><br>

### **IoC 컨테이너**

<span style="color:red">**컨테이너**</span>는 보통 객체의 생명 주기를 관리, 생성된 인스턴스들에게 추가적인 기능을 제공하도록 하는 것이다.<br>

스프링 프레임워크도 객체를 생성하고 관리하고 의존성을 관리해주는 IoC 컨테이너(=스프링 컨테이너)가 있다.<br>

인스턴스 생성부터 소멸까지의 인스턴스 생명 주기 관리를 개발자가 아닌 컨테이너가 대신 해준다.<br>
객체 관리 주체가 프레임워크(Container)가 되기 때문에 개발자는 로직에 집중할 수 있는 장점이 있다.<br>

* IoC 컨테이너는 객체의 생성을 책임지고, 의존성을 관리한다.
* POJO의 생성, 초기화, 서비스, 소멸에 대한 권한을 가진다.
* 개발자들이 적접 POJO를 생성할 수 있지만 컨테이너에게 맡긴다.
* 개발자는 비즈니스 로직에 집중할 수 있다.
* 객체 생성 코드가 없으므로 TDD가 용이하다.

<br><br>

## **IoC의 분류**
<hr />

### DL(Dependency Lookup)과 DI(Dependency Injection)
* DL : 저장소에 저장되어 있는 Bean에 접근하기 위해 컨테이너가 제공하는 API를 이용하여 Bean을 Lookup하는 것
* DI : 각 클래스간의 의존관계를 빈 설정(Bean Definition) 정보를 바탕으로 컨테이너가 자동으로 연결해주는 것
  * Setter Injection (수정자 주입)
  * Constructor Injection(생성자 주입)
  * Method Injection (필드 주입)

<br>

DL 사용시 컨테이너 종속이 증가하기 때문에 주로 DI를 사용한다.

<br><br>

## **스프링 컨테이너 (= IoC 컨테이너)의 종류**
<hr />

스프링 컨테이너가 관리하는 객체를 <span style="color:red">**빈(Bean)**</span>이라고 하고, 이 빈드를 관리한다는 의미로 컨테이너를 <span style="color:red">**빈 팩토리(Bean Factory)**</span>라고 부른다.<br>

* 객체의 생성과 객체 사이의 런타임 관계를 DI 관점에서 볼 때 컨테이너를 BeanFactory라고 한다.
* BeanFactory에 여러가지 컨테이너 기능을 추가한 <span style="color:red">**ApplicationContext**</span>가 있다.

<br>

### **BeanFactory**

BeanFactory 계열의 인터페이스만 구현한 클래스는 단순히 컨테이너에서 객체를 생성하고 DI를 처리하는 기능만 제공한다.<br>

* Bean을 등록, 생성, 조회, 반환 관리를 한다.
* 팩토리 디자인 패턴을 구현한 것으로 BeanFactory는 빈을 생성하고 분배하는 책임을 지는 클래스다.
* Bean을 조회할 수 있는 getBean() 메서드가 정의되어 있다.
* 보통은 BeanFactory를 바로 사용하지 않고, 이를 확장한 ApplicationContext를 사용한다.

<br><br>

### **ApplicationContext**  

**ApplicationContext**는 **BeanFactory** 인터페이스를 상속 받은 인터페이스이다.<br>

BeanFactory는 스프링 컨테이너의 최상위 인터페이스이다. 스프링 빈을 관리하고 조회하는 역할을 한다. ApplicationContext는 BeanFactory + 부가 기능(국제화 기능, 환경 변수 관련 처리, 애플리케이션 이벤트, 리소스 조회)을 가진다.<br>

정확히는 스프링 컨테이너를 부를 때, BeanFactory, ApplicationContext를 구분해서 말하지만, BeanFactory를 직접적으로 사용하는 경우는 거의 없다. 왜냐하면 ApplicationContext가 BeanFactory의 모든 기능을 가지고 있기 때문이다.<br><br>

**BeanFactory와의 차이점**<br>
<span style="color:red">**BeanFactroy는 lazy-loading(지연로딩) 방식**</span>이다. 지연 로딩은 해당 Bean을 호출할 때에야 Bean이 인스턴스화 되는 것으로 많은 Bean이 등록되어 있는 경우에 필요한 Bean을 그 때 그 때 인스턴스화 해 pre-loading보다 속도가 빠르다.<br>

반면에 <span style="color:red">**ApplicationContext는 pre-loading(사전로딩) 방식**</span>이어서 해당 Bean에 대한 설정 등을 스프링 컨테이너에서 이미 로드했을 때 Bean이 인스턴스화 되고 ApplicationContext에서 Bean을 호출할 때 기존에 인스턴스화된 Bean을 호출한다. 많은 Bean이 등록되어 있는 시스템일수록 더더욱 강점을 보일 수 있다.<br>

<hr />
참고자료<br>
<a href="https://dev-coco.tistory.com/80">https://dev-coco.tistory.com/80</a><br>
<a href="https://pamyferret.tistory.com/24">https://pamyferret.tistory.com/24</a><br>