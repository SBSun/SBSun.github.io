---
title: "[Spring Boot] Mocktio 기반의 단위 테스트"
excerpt: "Mocktio Test Framework에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring Boot, Mockito]

permalink: /spring/springboot-mockito/

toc: true
toc_sticky: true

date: 2023-01-30
last_modified_at: 2023-01-30

--- 
## **Mockito 소개 및 사용법**
<hr/>

<span style="font-size:120%">**Mockito란**</span><br>

Mockito는 <span style="color:red">**개발자가 동작을 직접 제어할 수 있는 가짜(Mock) 객체를 지원하는 테스트 프레임워크**</span>이다.<br>

일반적으로 Spring으로 웹 애플리케이션을 개발하면, 여러 객체들 간의 의존성이 생긴다. 이러한 의존성은 단위 테스트 작성을 어렵게 하는데, 이를 해결하기 위해 가짜 객체를 주입시켜주는 Mockito 라이브러리를 활용할 수 있다. Mockito를 활용하면 가짜 객체에 원하는 결과를 Stub하여 단위 테스트를 진행할 수 있다.<br>

물론 Mock을 하지 않아도 된다면 하지 않는 것이 좋다.<br><br>

<span style="font-size:120%">**Mockito 사용법**</span><br>

**1. Mock 객체 의존성 주입**<br>

Mockito에서 Mock(가짜) 객체의 의존성 주입을 위해서는 크게 3가지 어노테이션이 사용된다.<br>
* `@Mock` : Mock 객체를 만들어 반환해주는 어노테이션
* `@Spy` : Stub하지 않은 메서드들은 원본 메서드 그대로 사용하는 어노테이션
* `@InjectMocks` : `@Mock` 또는 `@Spy`로 생성된 가짜 객체를 자동으로 주입시켜주는 어노테이션

예를 들어 UserController에 대한 단위 테스트를 작성하고자 할 때, UserService를 사용하고 있다면 `@Mock` 어노테이션을 통해 가짜 UserService를 만들고, `@InjectMocks`를 통해 UserController에 이를 주입시킬 수 있다.<br><br>

**2. Stub로 결과 처리**<br>

의존성이 있는 객체는 <span style="color:red">**가짜 객체(Mock)를 주입하여 어떤 결과를 반환하라고 정해진 답변을 준비**</span>시켜야 한다. Mockito에서는 다음과 같은 stub 메서드를 제공한다.<br>
* `doReturn()` : Mock 객체가 특정한 값을 반환해야 하는 경우
* `doNothing()` : Mock 객체가 아무 것도 반환하지 않는 경우(void)
* `doThrow()` : Mock 객체가 예외를 발생시키는 경우

<br>

**3. Mockito와 JUnit의 결합**<br>

Mockito도 테스트 프레임워크이기 때문에 JUnit과 결합되기 위해서는 별도의 작업이 필요하다.<br>

기존의 JUnit4에서는 Mockito를 활용하기 위해서는 클래스 어노테이션으로 `@RunWith(MockitoJUnitRunner.class)`를 붙여주어야 연동이 가능했다. 하지만 SpringBoot 2.2버전 부터 공식적으로 JUnit5를 지원함에 따라, 이제부터는 `@ExtendWith(MockitoExtension.class)`를 사용해야 결합이 가능하다.<br><br>

## **Spring 컨트롤러 단위 테스트 작성 예시**
<hr/>

다음과 같은 회원 가입 API에 대한 단위 테스트를 작성해야 한다.<br>

``` java
@RestController
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping("/user/signUpUser")
    public UserResponseDto.Info signUpUser(UserRequestDto.SignUp signUp){
        return userService.signUp(signUp, Authority.USER);
    }
}
```

**단위 테스트 작성 준비**<br>

JUnit5와 Mockito를 연동하기 위해서는 `@ExtendWith(MockitoExtension.class)`를 사용해야 한다. 이를 클래스 어노테이션으로 붙여 테스트 클래스를 작성할 수 있다.<br>

``` java
@ExtendWith(MockitoExtension.class)
public class UserControllerTest {
}
```
<br>
이제 의존성 주입을 해주어야 한다. 먼저 테스트 대상인 UserController에는 가짜 객체 주입을 위한 `@InjectMocks`를 붙여주어야 한다. 그리고 UserService에는 가짜 Mock 객체 생성을 위해 `@Mock` 어노테이션을 붙여주면 된다.<br>

``` java
@ExtendWith(MockitoExtension.class)
public class UserControllerTest {
    @InjectMocks
    private UserController userController;

    @Mock
    private UserService userService;
}
```
<br>
컨트롤러를 테스트하기 위해서는 HTTP 호출이 필요하다. 일반적인 방법으로는 HTTP 호출이 불가능하므로 스프링에서는 이를 위핸 MockMvc를 제공하고 있다. MockMvc는 다음과 같이 생성할 수 있다.

``` java
@ExtendWith(MockitoExtension.class)
class UserControllerTest {
    @InjectMocks
    private UserController userController;

    @Mock
    private UserService userService;

    private MockMvc mockMvc;

    @BeforeEach
    public void init() {
        mockMvc = MockMvcBuilders.standaloneSetup(userController).build();
    }
}
```
<br>

**회원가입 성공 테스트**<br>

우선 회원가입 요청을 보내기 위해서는 UserRequestDto.SignUp 객체 1개와 userService의 signUp에 대한 stub이 필요하다. 이러한 준비 작업을 해주면 **given** 단계에 다음과 같은 테스트 코드가 작성된다.<br>

``` java
@DisplayName("회원 가입 성공")
@Test
void signUpSuccess() throws Exception{
    // given
    UserRequestDto.SignUp request = signUpRequest();
    UserResponseDto.Info response = userResponse();

    doReturn(response).when(userService)
            .signUp(any(UserRequestDto.SignUp.class));
}

private UserRequestDto.SignUp signUpRequest(){
    return UserRequestDto.SignUp.builder()
            .id("test@test.test")
            .password("test Password")
            .name("test Name")
            .address("test Address")
            .build();
}

private UserResponseDto.Info userResponse(){
    return UserResponseDto.Info.builder()
            .user_id("test@test.test")
            .password("test Password")
            .name("test Name")
            .address("test Address")
            .build();
}
```
<br>
HTTP 요청을 보내면 Spring은 내부에서 MessageConverter를 사용해 Json String을 객체로 변환한다.<br>

그런데 이것은 Spring 내부에서 진행되므로, API로 전달하는 파라미터인 UserRequestDto.SignUp을 조작할 수 없다.<br> 

그래서 UserRequestDto.SignUp 클래스 타입이라면 어떠한 객체도 처리할 수 있도록 `any()` 메서드가 사용되었다. `any()`를 사용할 때에는 특정 클래스의 타입을 지정해주는 것이 좋다.<br>

그 다음 **when** 단계에서는 mockMvc에 데이터와 함께 Post 요청을 보내야 한다. 요청 정보는 mockMvc의 perform에서 작성 가능한데, 요청 정보에는 `MockMvcRequestBuilders`가 사용되며 요청 메소드 종류, 내용, 파라미터 등을 설정할 수 있다. 보내는 데이터는 객체가 아닌 문자열이여야 하므로 `Gson`을 사용해 변환하였다.<br>

``` java
@DisplayName("회원 가입 성공")
@Test
void signUpSuccess() throws Exception{
    // given
    UserRequestDto.SignUp request = signUpRequest();
    UserResponseDto.Info response = userResponse();

    doReturn(response).when(userService)
            .signUp(any(UserRequestDto.SignUp.class));

    //when
    ResultActions resultActions = mockMvc.perform(
            MockMvcRequestBuilders.post("/users/signUpUser")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(new Gson().toJson(request))
    );
}
```
<br>

마지막으로 호출된 결과를 검증하는 **then** 단계에서는 회원가입 API 호출 결과로 200 Response와 응답 결과를 검증해야 한다. 응답 검증 시에는 jsonPath를 이용해 해당 json 값이 존재하는지 확인할 수도 있다.

``` java
@DisplayName("회원 가입 성공")
@Test
void signUpSuccess() throws Exception {
    // given
    UserRequestDto.SignUp request = signUpRequest();

    doReturn(userResponse()).when(userService)
            .signUp(any(UserRequestDto.SignUp.class));

    // when
    ResultActions resultActions = mockMvc.perform(
            MockMvcRequestBuilders.post("/user/signUpUser")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(new Gson().toJson(request))
    );

    // then
    MvcResult mvcResult = resultActions.andExpect(status().isOk()).andReturn();

    UserResponseDto.Info response = new Gson().fromJson(mvcResult.getResponse().getContentAsString(), UserResponseDto.Info.class);
    assertThat(response.getUser_id()).isEqualTo("test@test.test");
}
```
<br>

## **Spring 서비스 계층 단위 테스트 작성 예시**
<hr/>

사용자 회원가입을 위한 서비스는 다음과 같이 구현되어 있다.<br>

``` java 
@Service
@RequiredArgsConstructor
public class UserService{
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    //회원가입
    public UserResponseDto.Info signUp(UserRequestDto.SignUp signUp)
    {
        User user = User.builder()
                .id(signUp.getId())
                .password(passwordEncoder.encode(signUp.getPassword()))
                .name(signUp.getName())
                .address(signUp.getAddress())
                .authority(Authority.USER.getValue())
                .build();

        return UserResponseDto.Info.of(userRepository.save(user));
    }
}
```
<br>

**단위 테스트 작성 준비**<br>

``` java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    @InjectMocks
    private UserService userService;

    @Mock
    private UserRepository userRepository;

    @Spy
    private PasswordEncoder passwordEncoder;
}
```

PasswordEncoder에 `@Spy`를 사용하였다. 회원가입을 할 때 실제로 사용자 비밀번호를 암호화해야 하므로, `@Spy`를 사용했다.<br><br>

**회원가입 성공 테스트**<br>

``` java
@DisplayName("회원 가입")
@Test
void signUp(){
    // given
    UserRequestDto.SignUp request = signUpRequest();
    PasswordEncoder encoder = new BCryptPasswordEncoder();

    User user = User.builder()
            .id(request.getId())
            .password(encoder.encode(request.getPassword()))
            .name(request.getName())
            .address(request.getAddress())
            .authority(Authority.USER.getValue())
            .build();

    doReturn(user).when(userRepository)
            .save(any(User.class));

    // when
    UserResponseDto.Info info = userService.signUp(request);

    // then
    assertThat(user.getId()).isEqualTo(request.getId());
    assertThat(encoder.matches(request.getPassword(), info.getPassword())).isTrue();

    // verify
    verify(userRepository, times(1)).save(any(User.class));
    verify(passwordEncoder, times(1)).encode(any(String.class));
}
```

이번엔 추가적으로 Mockito의 `verify()`를 사용해봤다. `verify`는 <span style="color:red">**Mock된 객체의 특정 메서드가 호출된 횟수를 검증**</span>할 수 있다. 위에서는 passwordEncoder의 `encode` 메서드와 userRepository의 `save` 메서드가 각각 1번만 호출되었지를 검증하기 위해 사용했다.<br><br>

## **Spring 레포지토리 계층 단위 테스트 작성 예시**
<hr/>

사용자 회원가입을 위한 JPA 레포지토리 인터페이스는 다음과 같이 구현되어 있다.<br>

``` java
@Repository
public interface UserRepository extends JpaRepository<User, String> {
}
```
<br>

**`@DataJpaTest`**<br>

Spring Boot는 JPA 레포지토리를 손쉽게 테스트할 수 있는 `@DataJpaTest` 어노테이션을 제공한다. 기본적으로 인메모리 데이터베이스인 H2를 기반으로 테스트용 데이터베이스를 구축하며, 테스트가 끝나면 트랜잭션 롤백을 해준다.<br>

레포지토리 계층은 실제 DB와 통신없이 단순 모킹하는 것은 의미가 없으므로 직접 데이터베이스와 통신하는 `@DataJpaTest`를 사용한다.<br>

**`사용자 추가 테스트`**<br>

``` java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;

    @DisplayName("사용자 추가")
    @Test
    void addUser(){
        // given
        User user = user();

        // when
        User savedUser = userRepository.save(user);

        // then
        assertThat(savedUser.getId()).isEqualTo(user.getId());
        assertThat(savedUser.getPassword()).isEqualTo(user.getPassword());
        assertThat(savedUser.getName()).isEqualTo(user.getName());
        assertThat(savedUser.getAddress()).isEqualTo(user.getAddress());
        assertThat(savedUser.getAuthority()).isEqualTo(user.getAuthority());
    }

    private User user() {
        return User.builder()
                .id("test@test.test")
                .password("test Password")
                .name("test Name")
                .address("test Address")
                .authority(Authority.USER.getValue())
                .build();
    }
}
```

위 테스트에서 클래스 어노테이션으로 `@DataJpaTest`만 추가하여 실행해 보았는데 아래와 같은 에러가 발생했다.<br>

``` 
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'dataSourceScriptDatabaseInitializer' defined in class path resource [org/springframework/boot/autoconfigure/sql/init/DataSourceInitializationConfiguration.class]: Unsatisfied dependency expressed through method 'dataSourceScriptDatabaseInitializer' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dataSource': Invocation of init method failed; nested exception is java.lang.IllegalStateException: Failed to replace DataSource with an embedded database for tests. If you want an embedded database please put a supported one on the classpath or tune the replace attribute of @AutoConfigureTestDatabase.
```

에러내용을 검색해보니 **dataSourceScriptDatabaseInitializer**인 dataSource 빈을 생성하는데 에러가 발생하고, <span style="color:red">**Failed to replace DataSource with an embedded database for tests**</span> DataSource를 내장 데이터베이스로 교체하는데에 실패했다고 한다.<br>

`@DataJpaTest`는 기본적으로 <span style="color:red">**내장된 메모리 데이터베이스**</span>를 이용해 테스트를 실행해주는데 해당 프로젝트는 MySQL DB에 연결되어 있었기 때문에 내장된 메모리 DB가 아닌 MySQL DB로 테스트를 진행하여 발생한 문제였다.<br>

MySQL로 테스트를 진행하기 위해서는 **AutoConfigureTestDatabase** 에서 설정을 **EmbeddedDatabase**를 사용하지 않게 바꿔준다.<br>

``` java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class UserRepositoryTest {
}
```

Replace 값을 NONE으로 설정한다. `Replace.NONE` 일 경우 EmbeddedDatabase를 찾아 설정하지 않고 기존 애플리케이션에서 사용한 DataSource(저의 경우 MySQL)가 등록되게 된다.
<hr/>

참고자료<br>
<a href="https://mangkyu.tistory.com/145">https://mangkyu.tistory.com/145</a><br>
<a href="https://charliezip.tistory.com/21">https://charliezip.tistory.com/21</a><br>

