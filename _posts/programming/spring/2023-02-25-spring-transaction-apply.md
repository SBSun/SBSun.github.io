---
title: "[Spring] 선언적 Transaction 적용"
excerpt: "Spring에서 선언적 Transaction을 적용해보자."

categories:
  - Spring
tags:
  - [Spring]

published: true

permalink: /spring/transaction-apply/

toc: true
toc_sticky: true

date: 2023-02-25
last_modified_at: 2023-02-25

--- 

<a href="https://sbsun.github.io/spring/transaction-detail-setting/">Spring Transaction의 세부 설정</a>을 알아봤으니 이번에는 직접 Transaction을 적용해보자.<br><br>

## **@Transactional 어노테이션(선언적 트랜잭션)**
<hr />

### **@Transactional  어노테이션**

Spring에서는 클래스나 인터페이스 또는 메서드에 부여할 수 있는 `@Transactional`이라는 어노테이션을 제공한다.<br>
이 어노테이션이 붙으면 스프링은 <span style="color:red">**해당 타깃을 포인트 컷의 대상으로 자동 등록하여 트랜잭션 관리 대상**</span>이 된다.<br>
즉, 이 어노테이션을 통해 <span style="color:red">**포인트 컷에 등록하고 트랜잭션 속성을 부여**</span>하는 것이다.<br>

이렇듯 AOP를 이용해 코드 외부에서 트랜잭션의 기능을 부여해주고 속성을 지정할 수 있게 해주는 것을 선언적 트랜잭션이라고 한다.<br><br>

### **@Transactional의 롤백 처리**

Java에는 체크 예외(Checked Exception)와 언체크 예외(Unchecked Exception)가 있다.<br>
두 가지 예외 종류를 구분하는 것이 중요한 이유는 트랜잭션 롤백 범위가 다르기 때문이다. <br>

체크 예외란 Exception 클래스의 하위 클래스이며, 언체크 예외란 Exception 하위의 RuntimeException 하위의 예외이다.<br>

스프링의 선언적 트랜잭션 안에서 예외가 발생했을 때, 해당 예외가 언체크 예외(런타임 예외)라면 자동적으로 롤백이 발생한다. 트랜잭션의 `rollbackFor` 속성의 기본 값에 `RuntimeException`이 포함되어 있기 때문에 언체크 예외는 RuntimeException을 상속받은 클래스이기 때문에 자동으로 롤백한다.<br>
체크 예외를 롤백시키기 위해서는 `rollbackFor` 속성으로 해당 체크 예외를 지정해주어야 한다.<br>

스프링의 트랜잭션이 언체크 예외나 에러 만을 롤백 대상으로 보는 이유는 해당 예외들이 복구 가능성이 없는 예외들이므로 별도의 `try-catch`나 `throw`를 통해 처리를 강제하지 않기 때문이다.<br><br>

### **@Transactional의 대체 정책(Fallback Policy)**
만약 모든 메서드에 `@Transactional`이 붙어있으면 메서드가 더러워진다. 그래서 스프링은 메서드 외에도 클래스와 인터페이스에 어노테이션을 붙일 수 있도록 하고 있다.<br>

그리고 트랜잭션 어노테이션을 적용할 때 <span style="color:red">**타겟 메서드, 타켓 클래스, 선언 메서드, 선언 타입(클래스 or 인터페이스) 순으로**</span>`@Transactional`<span style="color:red">**이 적용되었는지 차례로 확인하고, 가장 먼저 발견되는 속성 정보를 사용**</span>한다.<br>

이를 4단계의 대체 정책(fallback policy)라고 부르며, 이를 통해 어노테이션을 최소화하는 동시에 세밀한 제어를 해줄 수 있다.<br><br>

## **Spring에서 트랜잭션의 사용법**
<hr />

### **1. 비지니스 로직과의 결합**

트랜잭션 중구난방으로 적용하는 것은 좋지 않다. 대신 특정 계층의 경계를 트랜잭션 경계와 일치시키는 것이 좋은데, 일반적으로 <span style="color:red">**비지니스 로직을 담고 있는 서비스 계층의 메서드와 결합**</span>시키는 것이 좋다.<br>

이유는 데이터 저장 계층(Repository)으로부터 읽어온 데이터를 사용하고 변경하는 등의 작업을 하는 곳이 서비스 계층이기 때문이다. <br>

``` java
@Service
@RequiredArgsConstructor
@Transactional
public class UserService{
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public Long signUp(UserRequestDto.SignUp signUp) {...}
}
```

서비스 계층을 트랜잭션의 시작과 종료 경계로 정했다면, 테스트와 같은 특별한 이유가 아니고는 다른 계층이나 모듈에서 DAO에 직접 접근하는 것은 차단해야 한다.<br>

트랜잭션은 보통 서비스 계층의 메서드 조합을 통해 만들어지기 때문에, DAO가 제공하는 주요 기능은 서비스 계층에 위임 메서드를 만들어줄 필요가 있다. 그리고 가능하면 다른 모듈의 DAO에 접근할 때는 서비스 계층을 거치도록 하는 것이 바람직하다.<br><br>

### **2. 읽기 전용 트랜잭션의 공통화**

클래스 레벨에는 공통적으로 적용되는 읽기 전용 트랜잭션 어노테이션을 선언하고, <span style="color:red">**추가나 삭제 또는 수정이 있는 작업에는 쓰기가 가능하도록 별도로**</span> `@Transactional` <span style="color:red">**어노테이션을 선언**</span>하는 것이 좋다. 이를 체감하기는 힘들겠지만 약간의 성능적인 이점을 얻을 수 있다.

``` java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserService{
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public UserResponseDto.Info getUserInfo(String email) {...}

    public List<UserResponseDto.Info> findAllUser() {...}

    @Transactional
    public Long signUp(UserRequestDto.SignUp signUp) {...}

    @Transactional
    public void editUser(UserRequestDto.Edit edit) {...}

    @Transactional
    public Long deleteUser() {...}
}
```

<br>

### **3. 테스트의 롤백**

트랜잭션 어노테이션을 테스트 클래스에 추가하면 <span style="color:red">**테스트의 DB 커밋을 롤백해주는 기능**</span>이 있다.<br>

DB와 연동되는 테스트를 할 때에는 DB의 상태와 데이터가 상당히 중요하다. 하지만 문제는 테스트에서 DB에 쓰기 작업을 하면 DB의 데이터가 바뀌는 것인데, 트랜잭션 어노테이션을 테스트에 활용하면 테스트를 진행하는 동안에 조작한 데이터를 모두 롤백하고 테스트를 진행하기 전의 상태로 만들어준다.<br><br>

어떠한 경우에도 커밋을 하지 않기 때문에 테스트의 성공 여부가 상관이 없으며 심지어 예외가 발생해도 어떠한 문제가 생기지 않는다. 강제로 롤백시키도록 설정되어 있기 때문이다.<br>

``` java
@Transactional
@ExtendWith(SpringExtension.class)
@DataJpaTest
class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;

    @Test
    void findByEmailAndPw() {
        final User user = User.builder()
                .email("email")
                .password("password")
                .name("name")
                .address("address")
                .authority(Authority.USER.getValue())
                .build();

        userRepository.save(user);

        assertThat(userRepository.findAll().size()).isEqualTo(1);
    }
}
```

테스트의 작업을 그대로 DB에 반영하고 싶다면 `@Rollback(false)`를 사용해주면 된다.<br>
`@Rollback`은 메서드에만 적용 가능하므로, 클래스 레벨에 부여하기를 원한다면 `@TransactionConfiguration(defaultRollback=false)`를 사용하고, 롤백을 원하는 메서드에 `@Rollback(true)`를 사용하면 된다.<br>

여기서 **auto_increment**나 **sequence** 등에 의해 증가된 값은 롤백이 되지 않는다.<br>
그렇기 때문에 테스트를 위해서는 별도의 DB로 연결을 하거나 또는 H2와 같은 휘발성 DB를 사용하는 것이 좋다.<br>

<hr />
참고자료<br>
<a href="https://mangkyu.tistory.com/170">https://mangkyu.tistory.com/170</a><br>