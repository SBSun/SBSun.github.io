---
title: "[Spring Boot] Spring Security + OAuth2 소셜 로그인 구현 (2) - OAuth 2.0 로그인 관련 클래스 구현"
excerpt: "OAuth2 로그인 관련 클래스들을 구현 해보자."

categories:
  - Spring
tags:
  - [Spring Boot, OAuth]

published: true

permalink: /spring/security-oauth2-login-class/

toc: true
toc_sticky: true

date: 2023-02-27
last_modified_at: 2023-02-27

--- 

## **OAuth2UserInfo 추상 클래스 및 자식 클래스**
<hr />

### **OAuth2UserInfo**

<br>

<span style="color:red">**소셜 타입별로 유저 정보를 가지는 추상클래스**</span><br>
OAuth2UserInfo 추상클래스를 상속받아 각 소셜 타입의 유저 정보 클래스를 구현한다.<br>

``` java
public abstract class OAuth2UserInfo {
    protected Map<String, Object> attributes;

    public OAuth2UserInfo(Map<String, Object> attributes){
        this.attributes = attributes;
    }

    // 소셜 식별 값 : 구글 - "sub", 카카오 - "id", 네이버 - "id"
    public abstract String getId();

    public abstract String getNickname();
}
```

현재 프로젝트에는 사용자의 이메일과 이름만 가져오면 되므로 `getId()`와 `getNickname()` 메서드만 생성했다.<br>

각 소셜에서 제공하는 정보 중에 사용하고 싶은 정보가 더 있다면 추가해서 사용하면 된다.<br><br>

### **NaverOAuth2UserInfo**
<br>

``` java
public class NaverOAuth2UserInfo extends OAuth2UserInfo{

    public NaverOAuth2UserInfo(Map<String, Object> attributes) {
        super(attributes);
    }

    @Override
    public String getId() {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        if (response == null) {
            return null;
        }
        return (String) response.get("id");
    }

    @Override
    public String getNickname() {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        if (response == null) {
            return null;
        }

        return (String) response.get("nickname");
    }
}
```

Naver의 경우, attributes를 받았을 때 바로 유저 정보가 있는 것이 아니라 `response` Key로 한 번 감싸져있기 때문에, `get("response")`로 꺼낸 후 사용할 정보 Key로 꺼내서 사용해야 한다.<br><br>

### **KakaoOAuth2UserInfo**
<br>

``` java
public class KakaoOAuth2UserInfo extends OAuth2UserInfo{

    public KakaoOAuth2UserInfo(Map<String, Object> attributes) {
        super(attributes);
    }

    @Override
    public String getId() {
        return String.valueOf(attributes.get("id"));
    }

    @Override
    public String getNickname() {
        Map<String, Object> account = (Map<String, Object>) attributes.get("kakao_account");
        Map<String, Object> profile = (Map<String, Object>) account.get("profile");

        if (account == null || profile == null) {
            return null;
        }

        return (String) profile.get("nickname");
    }
}
```

카카오는 네이버와 다르게 유저 정보가 `kakao_account.profile`로 2번 감싸져있는 구조이다.<br>
따라서 `get()`을 두 번 사용하여 데이터를 꺼낸 후 사용하고 싶은 정보의 Key로 꺼내서 사용하면 된다.<br><br>

### **GoogleOAuth2UserInfo**
<br>

``` java
public class GoogleOAuth2UserInfo extends OAuth2UserInfo{

    public GoogleOAuth2UserInfo(Map<String, Object> attributes) {
        super(attributes);
    }

    @Override
    public String getId() {
        return (String) attributes.get("sub");
    }

    @Override
    public String getNickname() {
        return (String) attributes.get("name");
    }
}
```

구글은 네이버, 카카오와 달리 유저 정보가 감싸져 있지 않기 때문에, 바로 `get()`으로 유저 정보 Key를 사용해서 꺼내면 된다.<br><br>

## **OAuth DTO 클래스 - OAuthAttributes**
<hr />

### **SocialType**

**OAuthAttributes** 클래스를 구현하기 전에 소셜을 구분하기 위한 Enum을 구현하자.<br>

``` java
public enum SocialType {
    NAVER, KAKAO, GOOGLE
}
```
<br>

### **OAuthAttributes**

**OAuthAttributes** 클래스는 각 소셜에서 받아오는 데이터가 다르므로, 소셜별로 받는 데이터를 분기 처리하는 DTO 클래스이다.<br>

**전체 코드**<br>
``` java
@Getter
public class OAuthAttributes {

    private String nameAttributeKey;       // OAuth2 로그인 진행 시 키가 되는 필드 값, PK와 같은 의미
    private OAuth2UserInfo oAuth2UserInfo; // 소셜 타입별 로그인 유저 정보

    @Builder
    public OAuthAttributes(String nameAttributeKey, OAuth2UserInfo oAuth2UserInfo) {
        this.nameAttributeKey = nameAttributeKey;
        this.oAuth2UserInfo = oAuth2UserInfo;
    }

    public static OAuthAttributes of(SocialType socialType, String userNameAttributeName, Map<String, Object> attributes){

        if(socialType == SocialType.NAVER){
            return ofNaver(userNameAttributeName, attributes);
        }
        if(socialType == SocialType.KAKAO){
            return ofKakao(userNameAttributeName, attributes);
        }

        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes){
        return OAuthAttributes.builder()
                .nameAttributeKey(userNameAttributeName)
                .oAuth2UserInfo(new NaverOAuth2UserInfo(attributes))
                .build();
    }

    private static OAuthAttributes ofKakao(String userNameAttributeName, Map<String, Object> attributes){
        return OAuthAttributes.builder()
                .nameAttributeKey(userNameAttributeName)
                .oAuth2UserInfo(new KakaoOAuth2UserInfo(attributes))
                .build();
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes){
        return OAuthAttributes.builder()
                .nameAttributeKey(userNameAttributeName)
                .oAuth2UserInfo(new GoogleOAuth2UserInfo(attributes))
                .build();
    }

    public User toEntity(OAuth2UserInfo oauth2UserInfo){
        return User.builder()
                .email(oauth2UserInfo.getEmail())
                .name(oauth2UserInfo.getNickname())
                .authority(Authority.USER.getValue())
                .build();
    }
}
```

<br>

### **OAuthAttributes 부분 코드 설명**
<br>

**필드**<br>

``` java
private String nameAttributeKey;       // OAuth2 로그인 진행 시 키가 되는 필드 값, PK와 같은 의미
private OAuth2UserInfo oAuth2UserInfo; // 소셜 타입별 로그인 유저 정보
```
`String nameAttributeKey`<br>
OAuth2 로그인 진행 시 키가 되는 필드 값으로, PK와 같은 의미<br>

`OAuth2UserInfo oauth2UserInfo`<br>
소셜 타입별 로그인 유저 정보를 가진 **OAuth2UserInfo** 인스턴스를 가진다.<br><br>

**OAuthAttributes를 생성하는 of 메소드**<br>

``` java
public static OAuthAttributes of(SocialType socialType, String userNameAttributeName, Map<String, Object> attributes){

    if(socialType == SocialType.NAVER){
        return ofNaver(userNameAttributeName, attributes);
    }
    if(socialType == SocialType.KAKAO){
        return ofKakao(userNameAttributeName, attributes);
    }

    return ofGoogle(userNameAttributeName, attributes);
}

private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes){
    return OAuthAttributes.builder()
            .nameAttributeKey(userNameAttributeName)
            .oAuth2UserInfo(new NaverOAuth2UserInfo(attributes))
            .build();
}

private static OAuthAttributes ofKakao(String userNameAttributeName, Map<String, Object> attributes){
    return OAuthAttributes.builder()
            .nameAttributeKey(userNameAttributeName)
            .oAuth2UserInfo(new KakaoOAuth2UserInfo(attributes))
            .build();
}

private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes){
    return OAuthAttributes.builder()
            .nameAttributeKey(userNameAttributeName)
            .oAuth2UserInfo(new GoogleOAuth2UserInfo(attributes))
            .build();
}
```

`OAuthAttributes of()`<br>
**CustomOAuth2UserService**에서 파라미터들을 주입받아 **OAuthAttributes** 객체를 생성하는 메서드<br>
**파라미터로 들어온 SocialType 별로 분기 처리하여 각 소셜 타입에 맞게 OAuthAttributes를 생성해준다.**<br>

`ofNaver(), ofKakao(), ofGoogle()`<br>
소셜 타입 별로 나눠서 OAuthAttributes 빌드 시 OAuth2UserInfo 필드에 각 소셜 타입의 OAuth2UserInfo를 생성하여 빌드한다.<br><br>

**OAuthAttributes의 정보로 내 서비스 User를 생성하는 toEntity 메소드**<br>

``` java
public User toEntity(OAuth2UserInfo oauth2UserInfo){
    return User.builder()
            .email(oauth2UserInfo.getEmail())
            .name(oauth2UserInfo.getNickname())
            .authority(Authority.USER.getValue())
            .build();
}
```

이후에 CustomOAuth2UserService에서 DB에 저장할 User를 OAuth2UserInfo의 정보를 사용하여 빌버로 빌드 후 반환한다.<br><br>

## **OAuth2UserService를 커스텀한 CustomOAuth2UserService**
<hr />

``` java
@Slf4j
@Service
@RequiredArgsConstructor
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserRepository userRepository;

    private static final String NAVER = "naver";
    private static final String KAKAO = "kakao";

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        log.debug("CustomOAuth2UserService.loadUser() 실행 - OAuth2 로그인 요청 진입");

        /**
         * DefaultOAuth2UserService 객체를 생성하여, loadUser(userRequest)를 통하 DefaultOAuth2UserService 객체를 생성 후 반환
         * DefaultOAuth2UserService의 loadUser()는 소셜 로그인 API의 사용자 정보 제공 URI로 요청을 보내서
         * 사용자 정보를 얻은 후, 이를 통해 DefaultOAuth2User 객체를 생성 후 반환한다.
         */
        OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        /**
         * userRequest에서 registrationId 추출 후 registrationId로 SocialType 저장
         * http://localhost:8080/oauth2/authorization/kakao에서 kakao가 registrationId
         * userNameAttributeName은 이후에 nameAttributeKey로 설정된다.
         */
        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        SocialType socialType = getSocialType(registrationId);
        // OAuth2 로그인 시 키(PK)가 되는 값
        String userNameAttributeName = userRequest.getClientRegistration()
                .getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();
        // 소셜 로그인에서 API가 제공하는 userInfo의 Json 값(유저 정보들)
        Map<String, Object> attributes = oAuth2User.getAttributes();

        // socialType에 따라 유저 정보를 통해 OAuthAttributes 객체 생성
        OAuthAttributes extractAttributes = OAuthAttributes.of(socialType, userNameAttributeName, attributes);

        // getUser() 메소드로 User 객체 생성 후 반환
        User createdUser = getUser(extractAttributes);

        // DefaultOAuth2User 객체를 생성해서 반환
        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(createdUser.getAuthority())),
                attributes,
                extractAttributes.getNameAttributeKey()
        );
    }

    // registrationId로 SocialType 반환
    private SocialType getSocialType(String registrationId) {
        if(NAVER.equals(registrationId)) {
            return SocialType.NAVER;
        }
        if(KAKAO.equals(registrationId)) {
            return SocialType.KAKAO;
        }
        return SocialType.GOOGLE;
    }

    /**
     * SocialType과 attributes에 들어있는 소셜 로그인의 식별값 id를 통해 회원을 찾아 반환하는 메소드
     * 만약 찾은 회원이 있다면, 그대로 반환하고 없다면 saveUser()를 호출하여 회원을 저장한다.
     */
    @Transactional(readOnly = true)
    private User getUser(OAuthAttributes attributes) {
        User findUser = userRepository.findByEmail(attributes.getOAuth2UserInfo().getEmail())
                .orElse(null);

        if(findUser == null) {
            return saveUser(attributes);
        }
        return findUser;
    }

    /**
     * OAuthAttributes의 toEntity() 메소드를 통해 빌더로 User 객체 생성 후 반환
     * 생성된 User 객체를 DB에 저장
     */
    @Transactional
    private User saveUser(OAuthAttributes attributes) {
        User createdUser = attributes.toEntity(attributes.getOAuth2UserInfo());
        return userRepository.save(createdUser);
    }
}
```

<br>

## **OAuth2 로그인 성공 시 로직을 처리하는 OAuth2LoginSuccessHandler**
<hr />

**OAuth2 로그인이 성공한다면, OAuth2LoginSuccessHandler의 로직이 실행된다.**

``` java
@Slf4j
@Component
@RequiredArgsConstructor
public class OAuth2LoginSuccessHandler implements AuthenticationSuccessHandler {
    private final JwtTokenProvider jwtTokenProvider;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response
            , Authentication authentication) throws IOException, ServletException {

        log.debug("OAuth2 Login 성공");

        OAuth2User oAuth2User = (OAuth2User)authentication.getPrincipal();

        UserResponseDto.TokenInfo tokenInfo = jwtTokenProvider.createToken(authentication);
        response.addHeader(jwtTokenProvider.getAccessHeader(), jwtTokenProvider.BEARER + tokenInfo.getAccessToken());

        jwtTokenProvider.sendAccessAndRefreshToken(response, tokenInfo.getAccessToken(), tokenInfo.getRefreshToken());
    }
}
```

로그인에 성공하면 Token을 발급하여 헤더에 실어 보낸다.<br><br>

## **OAuth2 로그인 실패 시 로직을 처리하는 OAuth2LoginFailureHandler**
<hr />

**OAuth2 로그인이 실패한다면, OAuth2LoginFailureHandler의 로직이 실행된다.**


``` java
@Slf4j
@Component
public class OAuth2LoginFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response
            , AuthenticationException exception) throws IOException, ServletException {

        response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
        response.getWriter().write("소셜 로그인 실패! 서버 로그를 확인해주세요.");
        log.debug("소셜 로그인에 실패했습니다. 에러 메시지 : {}", exception.getMessage());
    }
}
```

400 에러를 Response에 설정해주고, 에러 메시지와 로그를 설정해준다.

<hr />
참고자료<br>
<a href="https://ksh-coding.tistory.com/66">https://ksh-coding.tistory.com/66</a><br>