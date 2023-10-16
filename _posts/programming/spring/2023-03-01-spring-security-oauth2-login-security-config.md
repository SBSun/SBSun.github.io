---
title: "[Spring Boot] Spring Security + OAuth2 소셜 로그인 구현 (3) - SecurityConfig 설정"
excerpt: "SecurityConfig를 설정 해보자."

categories:
  - Spring
tags:
  - [Spring Boot, OAuth]

published: true

permalink: /spring/security-oauth2-login-security-config/

toc: true
toc_sticky: true

date: 2023-03-01
last_modified_at: 2023-03-01

--- 

## **SecurityConfig 클래스**
<hr />

SecurityConfig 클래스는 <span style="color:red">**Spring Security을 설정**</span>하는 클래스라고 생각하면 된다.<br><br>

### **SecurityConfig 전체 코드**

``` java
@EnableWebSecurity
@Configuration
@AllArgsConstructor
public class SecurityConfiguration {
    private final UserDetailsService userDetailsService;
    private final JwtTokenProvider jwtTokenProvider;
    private final RedisTemplate redisTemplate;
    private final OAuth2LoginSuccessHandler oAuth2LoginSuccessHandler;
    private final OAuth2LoginFailureHandler oAuth2LoginFailureHandler;
    private final CustomOAuth2UserService customOAuth2UserService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .formLogin().disable() // FormLogin 사용 x
                // rest api이므로 basic auth 및 csrf 보안을 사용하지 않는다는 설정이다.
                .httpBasic().disable() 
                .csrf().disable()
                // JWT를 사용하기 때문에 세션을 사용하지 않는다는 설정
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()

                // URL별 권한 관리 옵션
                .authorizeRequests()
                .antMatchers("USER만 요청가능한 URL들").hasRole("USER")
                .anyRequest().permitAll()
                .and()

                // 소셜 로그인 설정
                .oauth2Login()
                .successHandler(oAuth2LoginSuccessHandler)
                .failureHandler(oAuth2LoginFailureHandler)
                .userInfoEndpoint().userService(customOAuth2UserService);

        //  JWT 인증을 위하여 직접 구현한 필터를 UsernamePasswordAuthenticationFilter 전에 실행하겠다는 설정이다.
        http.addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider, redisTemplate), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public static BCryptPasswordEncoder bCryptPasswordEncoder() {
        //  AuthenticationManagerBuilder에  패스워드 암호화를 위해 Spring Security에서 제공하는 BCryptPasswordEncoder를 추가
        return new BCryptPasswordEncoder();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
    }
}
```
<br><br>

### **SecurityConfig 부분 설명**

<br>

<span style="color:orange; font-size:110%">**@EnableWebSecurity**</span> <span style="font-size:110%">**어노테이션**</span><br>

해당 어노테이션 코드를 들어가보면,<br>

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import({ WebSecurityConfiguration.class, SpringWebMvcImportSelector.class, OAuth2ImportSelector.class,
		HttpSecurityConfiguration.class })
@EnableGlobalAuthentication
@Configuration
public @interface EnableWebSecurity {

	/**
	 * Controls debugging support for Spring Security. Default is false.
	 * @return if true, enables debug support with Spring Security
	 */
	boolean debug() default false;
}
```

**@Import**로 다음과 같이 Spring Security와 관련된 클래스들을 import 해주는 것을 알 수 있다.<br>
1. WebSecurityConfiguration
2. SpringWebMvcImportSelector
3. OAuth2ImportSelector
4. HttpSecurityConfiguration

따라서, <span style="color:orange">**@EnableWebSecurity**</span> 어노테이션은 **Spring Security를 활성화시키는 어노테이션**이다.<br><br><br>

<span style="font-size:110%">**filterChain 메서드 - Spring Security 설정**</span><br>

**Spring Security 5.7버전 이상부터는 Spring Security의 설정 시에 설정 메서드를 @Bean을 통해 빈으로 등록해서 컨테이너가 관리하도록 사용한다.**<br>
**(이전에는 빈 등록 대신 WebSecurityConfigurerAdapter를 상속받아 메서드를 Override 했었다.)**<br>

``` java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .formLogin().disable() // FormLogin 사용 x
            // rest api이므로 basic auth 및 csrf 보안을 사용하지 않는다는 설정이다.
            .httpBasic().disable() 
            .csrf().disable()
            // JWT를 사용하기 때문에 세션을 사용하지 않는다는 설정
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()

            // URL별 권한 관리 옵션
            .authorizeRequests()
            .antMatchers("USER만 요청가능한 URL들").hasRole("USER")
            .anyRequest().permitAll()
            .and()

            // 소셜 로그인 설정
            .oauth2Login()
            .successHandler(oAuth2LoginSuccessHandler)
            .failureHandler(oAuth2LoginFailureHandler)
            .userInfoEndpoint().userService(customOAuth2UserService);

    //  JWT 인증을 위하여 직접 구현한 필터를 UsernamePasswordAuthenticationFilter 전에 실행하겠다는 설정이다.
    http.addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider, redisTemplate), UsernamePasswordAuthenticationFilter.class);
    return http.build();
}
```

세부적인 보안 기능 설정 API를 제공하는 HttpSecurity를 파라미터로 받아서 HttpSecurity의 보안 기능 설정 API를 사용하여 최종적으로 여러 설정을 마친 SecurityFilterChain 객체를 반환하는 메서드이다.<br><br><br>

<span style="font-size:110%">**기타 설정**</span><br>

``` java
.formLogin().disable()
.httpBasic().disable() 
.csrf().disable()
.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
```

**1. formLogin().disable()**<br>
Spring Security에서는 아무 설정을 하지 않으면, 기본으로 formLogin 형식의 로그인을 제공한다.<br>
프로젝트에서는 자체 Login으로 로그인을 진행하기 때문에, 기본 방식을 disable했다.<br>

**2. httpBasic().disable()**<br>
프로젝트에서는 JWT 토큰을 사용한 로그인 방식이기 때문에 기본 설정인 httpBasic은 disable했다.<br>

**3. csrf().disable()**<br>
프로젝트에서는 REST API를 사용하여 서버에 인증 정보를 저장하지 않고, 요청 시 인증 정보(JWT 토큰, OAuth2)를 담아서 요청하므로 csrf를 disable했다.<br>

**4. sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)**<br>
JWT 토큰 방식을 사용하므로 세션은 사용하지 않기 때문에 세션 정책을 Stateless로 설정했다.<br><br><br>

<span style="font-size:110%">**URL별 인증/인가 설정**</span><br>

``` java
.authorizeRequests()
.antMatchers("USER만 요청가능한 URL들").hasRole("USER")
.anyRequest().permitAll()
```

**1. authorizeRequests()**<br>
인증/인가 설정 시 **HttpServletRequest**을 이용한다는 것을 의미한다.<br>

**2. antMatchers("URL").hasRole("USER")**<br>
**andMatchers의 파라미터에 설정한 URL은 모든 리소스의 접근을 USER 권한을 가졌을 때만 사용 가능하다는 의미다.**<br>

**3. anyRequest().permitAll()**<br>
모든 리소스(anyRequest)는 인증 없이 접근 가능하다는 의미이다.<br>
위에서부터 순차적으로 코드가 실행되어 HttpSecurity에 설정되기 때문에 위에 **andMatchers().hasRole()로 설정한 URL 이외의 모든 URL은 인증 없이도 사용 가능함을 의미한다.**<br><br><br>

<span style="font-size:110%">**소셜 로그인(OAuth2 Login) 설정**</span><br>

``` java
.oauth2Login()
.successHandler(oAuth2LoginSuccessHandler)
.failureHandler(oAuth2LoginFailureHandler)
.userInfoEndpoint().userService(customOAuth2UserService);
```

**1. oauth2Login()**<br>
Http Security의 OAuth2 로그인 관련 **OAuth2LoginConfigurer**를 반환하여 OAuth2 로그인에 관련된 다양한 기능을 사용할 수 있도록한다.<br>

**2. successHandler()**<br>
OAuth2 로그인이 성공했을 때 처리할 핸들러를 설정해준다.<br>

**3. failureHandler()**<br>
OAuth2 로그인이 실패했을 때 처리할 핸들러를 설정해준다.<br>

**4. userInfoEndpoint().userService()**<br>
**OAuth2 로그인의 로직을 담당하는 Service를 설정**할 수 있다.<br><br><br>

<span style="font-size:110%">**스프링 시큐리티에 커스텀 필터 추가 설정**</span><br>

``` java
 http.addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider, redisTemplate), UsernamePasswordAuthenticationFilter.class);
```

**1. addFilterBefore(A, B)**<br>
B 필터 이전에 A 필터가 동작하도록 하는 메서드이다.<br>
**JwtAuthenticationFilter**를 **UsernamePasswordAuthenticationFilter** 전으로 설정했다.<br>

<hr />
참고자료<br>
<a href="https://ksh-coding.tistory.com/70#recentEntries">https://ksh-coding.tistory.com/70#recentEntries</a><br>