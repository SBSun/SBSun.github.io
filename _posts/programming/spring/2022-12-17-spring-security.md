---
title: "[Spring] Spring Security"
excerpt: "Spring 기반의 애플리케이션의 보안을 담당하는 Spring Security 프레임워크를 알아보자"

categories:
  - Spring
tags:
  - [Spring]

permalink: /spring/spring-security/

toc: true
toc_sticky: true

date: 2022-12-17
last_modified_at: 2022-12-18
---

현재 진행하고 있는 Web 프로젝트에 Spring Security를 사용하기 위해 알아보게 되었다.
<br><br>

## Spring Security란?
<hr/>
Spring 기반의 <span style="color:red">애플리케이션 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크</span>이다.
<br><br>
Spring Security는 '인증'과 '권한'에 대한 부분을 <span style="color:red">Filter 흐름에 따라 처리</span>하고 있다.
<br><br>
Filter는 Dispatcher Servlet으로 가기 전에 적용되므로 가장 먼저 URL 요청을 받지만, <br>
Interceptor는 Dispatcher와 Controller 사이에 위치한다는 점에서 적용 시기의 차이가 있다.
<br><br>
Client (request) → <span style="color:red">Filter</span> → DispatcherServlet → <span style="color:red">Interceptor</span> →  Controller <br>
(실제로 Interceptor가 Controller로 요청을 위임하는 것은 아님, Interceptor를 거쳐서 가는 것)
<br><br>
Spring Security는 보안과 관련해서 체계적으로 많은 옵션을 제공해주기 때문에,<br>
개발자 입장에서는 일일이 보안관련 로직을 작성하지 않아도 된다는 장점이 있다.<br><br>



## 인증(Authorization)과 인가(Authentication)
<hr/>

회원이 존재하는 시스템이라면, 그에 따른 인증과 인가에 대한 처리를 해야 한다.<br>
* 인증(Authorization) : 해당 사용자가 본인이 맞는지를 확인하는 절차
* 인가(Authentication) : 해당 사용자가 요청하는 request를 실행할 수 있는 권한이 있는지
확인하는 절차.

<br>
Spring Security는 기본적으로 인증 절차를 거친 후에 인가 절차를 진행하게 되며, 인가 과정에서 해당 리소스에 대한 접근 권한이 있는지 확인하게 된다.<br><br>

Spring Security에서는 이러한 인증과 인가를 위해 Principal을 아이디로, Credential을 비밀번호로 사용하는 <span style="color:red">Credential 기반의 인증 방식을 사용</span>한다.
<br>
* Principal(접근 주체): 보호받는 Resource에 접근하는 대상
* Credential(비밀번호): Resource에 접근하는 대상의 비밀번호
<br><br>

## Spring Security Authentication Architecture
<hr/>

<img src="../../assets/images/posts/programming/spring/spring-security/spring-security-1.PNG">
<br><br>
1. 사용자가 로그인 정보와 함께 인증 요청을 한다. (Http Request)<br>
2. AuthenticationFilter가 요청을 가르채고, 가르챈 정보를 통해 UsernamePasswordAuthenticationToken(인증용 객체)을 생선한다.<br>
3. AuthenticationManager의 구현체인 ProviderManager에게 생성한 UsernamePasswordToken 객체를 전달한다.<br>
4. 다시 AuthenticationProvider에 UsernamePasswordAuthenticationToken 객체를 전달한다.<br>
5. 실제 데이터베이스에서 사용자 인증정보를 가져오는 UserDetailsService에 사용자 정보(아이디)를 넘겨준다.<br>
6. 넘겨받은 사용자 정보를 통해 DB에서 찾은 사용자 정보인 UserDetails 객체를 만든다.<br>
7. AuthenticationProvider는 UserDetails를 넘겨받고 사용자 정보를 비교한다.<br>
8. 인증이 완료되면 권한 등의 사용자 정보를 담은 Authentication 객체를 반환한다.<br>
9. 다시 최초의 AuthenticationFilter에 Authentication 객체가 반환된다.<br>
10. Authentication 객체를 SecurityContext에 저장한다.<br><br>
    
최종적으로 SecurityContextHolder는 세션 영역에 있는 SecurityContext에 Authentication 객체를 저장한다. <br>세션에 사용자정보를 저장한다는 것은 스프링 시큐리티가 전통적인 세션-쿠키 기반의 인증 방식을 사용한다는 것을 의미한다.<br><br>

## Spring Security 주요 모듈
<hr/>

<span style="color:blue; font-size:130%">**Authentication**</span><br>
Authentication는 현재 접근하는 주체의 정보와 권한을 담는 인터페이스이다.<br>
Authentication 객체는 Security Context에 저장되며, SecurityContextHolder를 통해 SecurityContext에 접근하고, <br>SecurityContext를 통해 Authentication에 접근할 수 있다.

```java
public interface Authentication extends Principal, Serializable {
    // 현재 사용자의 권한 목록을 가져옴
    Collection<? extends GrantedAuthority> getAuthorities();
    
    // credentials(주로 비밀번호)을 가져옴
    Object getCredentials();
    
    Object getDetails();
    
    // Principal 객체를 가져옴.
    Object getPrincipal();
    
    // 인증 여부를 가져옴
    boolean isAuthenticated();
    
    // 인증 여부를 설정함
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```
<br>

 <span style="color:blue; font-size:130%">**UsernamePasswordAuthenticationToken**</span><br>
UsernamePasswordAuthenticationToken은 Authentication을 implements한 AbstractAuthenticationToken의 하위 클래스로,<br> User의 ID가 Principal 역할을 하고, Password가 Credential의 역할을 한다. UsernamePasswordAuthenticationToken의 <br>첫 번째 생성자는 인증 전의 객체를 생성하고, 두번째 생성자는 인증이 완려된 객체를 생성한다.

```java
public abstract class AbstractAuthenticationToken implements Authentication, CredentialsContainer {}

public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
    // 주로 사용자의 ID에 해당함
    private final Object principal;
    // 주로 사용자의 PW에 해당함
    private Object credentials;
    
    // 인증 완료 전의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
		super(null);
		this.principal = principal;
		this.credentials = credentials;
		setAuthenticated(false);
	}
    
    // 인증 완료 후의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
		super(authorities);
		this.principal = principal;
		this.credentials = credentials;
		super.setAuthenticated(true); // must use super, as we override
	}
}
``` 
<br>

<span style="color:blue; font-size:130%">**AuthenticationManager**</span><br>
인증에 대한 부분은 SpringSecurity의 AuthenticatonManager를 통해서 처리하게 되는데, 실질적으로는 AuthenticationManager에 등록된 AuthenticationProvider에 의해 처리된다. 인증이 성공하면 2번째 생성자를 이용해 인증이 성공한(isAuthenticated=true) 객체를 생성하여 Security Context에 저장한다. 그리고 인증 상태를 유지하기 위해 세션에 보관하며, 인증이 실패한 경우에는 AuthenticationException를 발생시킨다.

```java
public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication) 
		throws AuthenticationException;
}
```
<br>

<span style="color:blue; font-size:130%">**AuthenticationProvider**</span><br>
AuthenticationProvider에서는 실제 인증에 대한 부분을 처리하는데, 인증 전의 Authentication객체를 받아서 인증이 완료된 객체를 반환하는 역할을 한다. 아래와 같은 AuthenticationProvider 인터페이스를 구현해서 Custom한 AuthenticationProvider을 작성해서 AuthenticationManager에 등록하면 된다.

```java
public interface AuthenticationProvider {
	// 인증 전의 Authenticaion 객체를 받아서 인증된 Authentication 객체를 반환
    Authentication authenticate(Authentication var1) throws AuthenticationException;

    boolean supports(Class<?> var1);
}
```
<br>

<span style="color:blue; font-size:130%">**ProviderManager**</span><br>
AuthenticationManager를 implements한 ProviderManager는 실제 인증 과정에 대한 로직을 가지고 있는 AuthenticaionProvider를 List로 가지고 있으며, ProividerManager는 for문을 통해 모든 provider를 조회하면서 authenticate 처리를 한다.

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware,
InitializingBean {
    public List<AuthenticationProvider> getProviders() {
		return providers;
	}
    public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		Authentication result = null;
		boolean debug = logger.isDebugEnabled();
        //for문으로 모든 provider를 순회하여 처리하고 result가 나올 때까지 반복한다.
		for (AuthenticationProvider provider : getProviders()) { ... }
    throw lastException;
  } 
}
```
<br>

<span style="color:blue; font-size:130%">**UserDetailsService**</span><br>
UserDetailsService 인터페이스는 UserDetails 객체를 반환하는 단 하나의 메소드를 가지고 있는데,<br> 일반적으로 이를 구현한 클래스의 내부에 UserRepository를 주입받아 DB와 연결하여 처리한다.

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```
<br>

<span style="color:blue; font-size:130%">**UserDetails**</span><br>
UserDetails 객체는 Authentication객체를 구현한 UsernamePasswordAuthenticationToken을 생성하기 위해 사용된다. <br>
UserDetails를 implements하여 이를 처리한다.

```java
public interface UserDetails extends Serializable {
    // 계정의 권한 목록을 리턴
    Collection<? extends GrantedAuthority> getAuthorities();
    // 계정의 비밀번호를 리턴
    String getPassword();
    // 계정의 고유한 값을 리턴( ex : DB PK값, 중복이 없는 이메일 값 )	
    String getUsername();
    // 계정의 만료 여부 리턴, true - 만료 안됨
    boolean isAccountNonExpired();
    // 계정의 잠김 여부 리턴, true - 잠기지 않음
    boolean isAccountNonLocked();
    // 비밀번호 만료 여부 리턴, true - 만료 안됨
    boolean isCredentialsNonExpired();
    // 계정의 활성화 여부 리턴, ture - 활성화 됨
    boolean isEnabled();
    
}
```
<br>

<span style="color:blue; font-size:130%">**SecurityContextHolder**</span><br>
SecurityContextHolder는 SecurityContext를 3가지의 저장 방식으로 나눠 관리한다.<br>
* MODE_THREADLOCAL : 스레드당 SecurityContext 객체를 할당한다. 기본값
* MODE_INHERITABLETHREADLOCAL : 메인 스레드와 자식 스레드에 관하여 동일한 SecurityContext를 유지한다.
* MODE_GLOBAL :  응용 프로그램에서 단 하나의 SecurityContext를 저장한다.<br><br>

<span style="color:blue; font-size:130%">**SecurityContext**</span><br>
SecurityContext는 Authentication 객체를 저장한다. TheadLocal에 저장되어 아무 곳에서나 참조 가능하다(Thread Safe)<br><br>

<span style="color:blue; font-size:130%">**GrantedAuthority**</span><br>
GrantAuthority는 현재 사용자(principal)가 가지고 있는 권한을 의미한다. ROLE_ADMIN나 ROLE_USER와 같이 ROLE_*의 형태로 사용하며, 보통 "roles" 이라고 한다. GrantedAuthority 객체는 UserDetailsService에 의해 불러올 수 있고, 특정 자원에 대한 권한이 있는지를 검사하여 접근 허용 여부를 결정한다.
<br>
 <hr/>
참고 자료
<br><a href="https://dev-coco.tistory.com/174" target="_blank">https://dev-coco.tistory.com/174</a>
<br><a href="https://webfirewood.tistory.com/115" target="_blank">https://webfirewood.tistory.com/115</a>
<br><a href="https://mangkyu.tistory.com/76" target="_blank">https://mangkyu.tistory.com/76</a>