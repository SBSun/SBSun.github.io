---
title: "[Spring] 스프링의 예외 처리 방법(ExceptionHandler, ControllerAdvice 등)"
excerpt: "스프링의 예외 처리 방법에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring]

published: true

permalink: /spring/exception-handling/

toc: true
toc_sticky: true

date: 2023-03-21
last_modified_at: 2023-03-21

--- 

## **스프링의 기본적인 예외 처리 방법**
<hr />

Spring은 만들어질 때(1.0)부터 <span style="color:red">**에러 처리를 위한 BasicErrorController**</span>를 구현해두었고, Spring Boot는 예외가 발생하면 기본적으로 `/error`로 <span style="color:red">**에러 요청을 다시 전달하도록 WAS 설정**</span>을 해두었다.<br>

참고로 이는 Spring Boot의 WebMvcAutoConfiguration을 통해 자동 설정이 되는 WAS의 설정이다. 여기서 요청이 `/error`로 다시 전달된다는 부분에 주목해야 한다. 일반적인 요청 흐름은 다음과 같이 진행된다.<br>

```
WAS(톰캣) -> 필터 -> 서블릿(디스패처 서블릿) -> 인터셉터 -> 컨트롤러
```

<br>

그리고 컨트롤러 하위에서 예외가 발생했을 때, 별도의 예외 처리를 하지 않으면 WAS까지 에러가 전달된다. 그러면 WAS는 애플리케이션에서 처리를 못하는 예외라 Exception이 올라왔다고 판단하고, 대응 작업을 진행한다.<br>

``` 
컨트롤러(예외 발생) -> 인터셉터 -> 서블릿(디스패처 서블릿) -> 필터 -> WAS(톰캣)
```

<br>

WAS는 스프링 부트가 등록한 예외 설정(/error)에 맞게 요청을 전달하는데, 흐름을 총 정리하면 다음과 같다.<br>

```
WAS(톰캣) -> 필터 -> 서블릿(디스패처 서블릿) -> 인터셉터 -> 컨트롤러
-> 컨트롤러(예외 발생) -> 인터셉터 -> 서블릿(디스패처 서블릿) -> 필터 -> WAS(톰캣)
-> WAS(톰캣) -> 필터 -> 서블릿(디스패처 서블릿) -> 인터셉터 -> 컨트롤러(BasicErrorController)
```

<br>

기본적인 예외 처리 방식은 <span style="color:red">**결국 에러 컨트롤러를 한번 더 호출**</span>하는 것이다. 그러므로 필터나 인터셉터가 다시 호출될 수 있는데, 이를 제어하기 위해서는 별도의 설정이 필요하다.<br>

서블릿은 **dispatcherType**으로 요청의 종류를 구분하는데, 일반적인 요청은 **REQUEST**이며 에러 요청은 **ERROR**이다.<br>

필터는 서블릿 기술이므로 필터 등록(FilterRegistrantionBean) 시에 호출될 **dispatcherType** 타입을 설정할 수 있고, 별도의 설정이 없다면 **REQUEST**일 경우에만 필터가 호출된다. 하지만 인터셉터는 스프링 기술이므로 **dispatcherType**을 설정할 수 없어 URI 패턴으로 처리가 필요하다.<br>

스프링 부트에서는 WAS까지 직접 제어하게 되면서 이러한 WAS의 에러 설정까지 가능해졌다. 또한 이는 <span style="color:red">**요청이 2번 생기는 것은 아니고, 1번의 요청이 2번 전달되는 것**</span>이다. 그러므로 클라이언트는 이러한 에러 처리 작업이 진행되었는지 알 수 없다.

<br><br>

### **BasicErrorController의 동작 및 에러 속성들**
<br>

BasicErrorController는 accept 헤더에 따라 에러 페이지를 반환하거나 에러 메세지를 반환한다. 에러 경로로는 기본적트로 `/error`로 정의되어 있으며 properties에서 `server.error.path`로 변경할 수 있다.<br>

``` java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

    private final ErrorProperties errorProperties;
    ...

    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        ...
    }

    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        ...
        return new ResponseEntity<>(body, status);
    }
    
    protected ErrorAttributeOptions getErrorAttributeOptions(HttpServletRequest request, MediaType mediaType) {
        ...
        return options;
    }

    ...
}
```

errorHtml()과 error()는 모두 getErrorAttributeOptions를 호출해 반환할 에러 속성을 얻는데, 기본적으로 <span style="color:red">**DefaultErrorAttributes**로부터 반환할 정보를 가져온다.</span> DefaultErrorAttributes는 전체 항목들에서 설정에 맞게 불필요한 속성들을 제거한다.<br>

* timestamp : 에러가 발생한 시간
* status : 에러의 HTTP 상태
* error : 에러 코드
* path : 에러가 발생한 URI
* exception : 최상위 예외 클래스의 이름(설정 필요)
* message : 에러에 대한 내용(설정 필요)
* errors : BindingExecption에 의해 생긴 에러 목록(설정 필요)
* trace : 에러 스택 트레이스(설정 필요)

``` json
{
    "timestamp": "2021-12-31T03:35:44.675+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/product/5000"
}
```

<br>

위는 기본 설정으로 받는 에러 응답인데, <span style="color:red">**클라이언트 입장에선 유용하지 못하다.**</span> 클라이언트는 "Item with id 5000 not found"라는 메세지와 함께 404 status로 에러 응답을 받으면 훨씬 유용할 것이다.

<br><br>

## **스프링이 제공하는 다양한 예외처리 방법**
<hr />

자바에서는 예외 처리를 위해 try-catch를 사용해야 하지만 try-catch를 모든 코드에 붙이는 것은 비효율적이다. Spring은 <span style="color:red">**에러 처리라는 공통 관심사(cross-cutting concerns)를 메인 로직으로부터 분리**</span>하는 다양한 예외 처리 방식을 고안하였고, 예외 처리 전략을 추상화한 **HandlerExceptionResolver** 인터페이스를 만들었다. <br>

대부분의 HandlerExceptionResolver는 발생한 Exception을 catch하고 HTTP 상태나 응답 메세지 등을 설정한다. 그래서 <span style="color:red">**WAS 입장에서는 해당 요청이 정상적인 응답인 것으로 인식**</span>된다.<br>

``` java
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, 
            HttpServletResponse response, Object handler, Exception ex);
}
```

<br>

위의 Object 타입인 handler는 예외가 발생한 컨트롤러 객체이다. 예외가 던져지면 디스패처 서블릿까지 전달되는데, 적합한 예외 처리를 위해 HandlerExceptionResolver 구현체들을 빈으로 등록해서 관리한다. 그리고 적용 가능한 구현체를 찾아 예외 처리를 하는데, 우선순위대로 아래의 4가지 구현체들이 빈으로 등록되어 있다.<br>

* DefaultErrorAttributes : 에러 속성을 저장하며 직접 에러를 처리하지는 않는다.
* ExceptionHandlerExceptionResolver : 에러 응답을 위한 Controller나 ControllerAdvice에 있는 ExceptionHandler를 처리한다.
* ResponseStatusExceptionResolver : HTTP 상태 코드를 지정하는 @ResponseStatus 또는 ResponseStatusException를 처리한다.
* DefaultHandlerExceptionResolver : 스프링 내부의 기본 예외들을 처리한다.

<br>

Spring은 아래와 같은 도구들로 ExceptionResolver를 동작시켜 에러를 처리할 수 있는데, 각각의 방식에 대해 자세히 알아보자.
1. ResponseStatus
2. ResponseStatusException
3. ExceptionHandler
4. ControllerAdvice, RestControllerAdvice

<br><br>

### **@ResponseStatus**

`@ResponseStatus`는 <span style="color:red">**에러 HTTP 상태를 변경**</span>하도록 도와주는 어노테이션이다.<br>
`@ResponseStatus`는 다음과 같은 경우들에 적용할 수 있다.

* Exception 클래스 자체
* 메서드에 `@ExceptionHandler`와 함께
* 클래스에 `@RestControllerAdvice`와 함께

<br>

예를 들어 우리가 만든 예외 클래스에 다음과 같이 `@ResponseStatus`로 응답 상태를 지정해줄 수 있다.

``` java
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class NoSuchElementFoundException extends RuntimeException {
  ...
}
```

<br>

그러면 ResponseStatusExceptionResolver가 지정해준 상태로 에러 응답이 내려가도록 처리한다.<br>

``` java
{
    "timestamp": "2021-12-31T03:35:44.675+00:00",
    "status": 404,
    "error": "Not Found",
    "path": "/product/5000"
}
```

<br>

하지만 에러 응답에서 볼 수 있듯이 이는 BasicErrorController에 의한 응답이다.<br>
즉, `@ResponseStatus`를 처리하는 ResponseStatusExceptionResolver는 WAS까지 예외를 전달시키며, 복잡한 WAS의 에러 요청 전달이 진행되는 것이다. 이러한 `@ResponseStatus`는 다음과 같은 한계점을 가지고 있다.<br>

때문에 개발자가 원하는대로 에러를 처리하는 것이 어렵다. 이러한 문제를 해결하기 위해서는 다른 방법을 사용해야 한다.

<br><br>

### **ResponseStatusException**

외부 라이브러리에서 정의한 코드는 우리가 수정할 수 없으므로 @ResponseStatus를 붙여줄 수 없다. Spring5에는 <span style="color:red">**@ResponseStatus의 프로그래밍적 대안**</span>으로써 <span style="color:red">**손쉽게 에러를 반환**</span>할 수 있는 ResponseStatusException가 추가되었다.<br>

ResponseStatusException는 HttpStatus와 함께 선택적으로 reason과 cause를 추가할 수 있고, 언체크 예외을 상속받고 있어 명시적으로 에러를 처리해주지 않아도 된다.<br>

``` java
@GetMapping("/product/{id}")
public ResponseEntity<Product> getProduct(@PathVariable String id) {
    try {
        return ResponseEntity.ok(productService.getProduct(id));
    } catch (NoSuchElementFoundException e) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Item Not Found");
    }
}
```

<br>

@ResponseStatus와 동일하게 예외가 발생하면 ResponseStatusExceptionResolver가 에러를 처리한다. ResponseStatusException를 사용하면 다음과 같은 이점을 누릴 수 있다.<br>

* 기본적인 예외 처리를 빠르게 적용할 수 있으므로 손쉽게 프로토타이핑할 수 있음
* HttpStatus를 직접 설정하여 예외 클래스와의 결합도를 낮출 수 있음
* 불필요하게 많은 별도의 예외 클래스를 만들지 않아도 됨
* 프로그래밍 방식으로 예외를 직접 생성하므로 예외를 더욱 잘 제어할 수 있음

<br>

하지만 그럼에도 불구하고 ResponseStatusException는 다음과 같은 한계점들을 가지고 있다. 이러한 이유로 API 에러 처리를 위해서는 @ExceptionHandler를 사용하는 방식이 더 많이 사용된다.<br>

* 직접 예외 처리를 프로그래밍하므로 일관된 예외 처리가 어려움
* 예외 처리 코드가 중복될 수 있음
* Spring 내부의 예외를 처리하는 것이 어려움
* 예외가 WAS까지 전달되고, WAS의 에러 요청 전달이 진행됨

<br><br>

### **@ExceptionHandler**

`@ExceptionHandler`는 <span style="color:red">**매우 유연하게 에러를 처리할 수 있는 방법**</span>을 제공하는 기능이다. @ExceptionHandler는 다음에 어노테이션을 추가함으로써 에러를 손쉽게 처리할 수 있다.

<br>

예를 들어 다음과 같이 <span style="color:red">**컨트롤러의 메소드에 @ExceptionHandler를 추가함으로써 에러를 처리**</span>할 수 있다. @ExceptionHandler에 의해 발생한 예외는 ExceptionHandlerExceptionResolver에 의해 처리가 된다.<br>

``` java
@RestController
@RequiredArgsConstructor
public class ProductController {

  private final ProductService productService;
  
  @GetMapping("/product/{id}")
  public Response getProduct(@PathVariable String id){
    return productService.getProduct(id);
  }

  @ExceptionHandler(NoSuchElementFoundException.class)
  public ResponseEntity<String> handleNoSuchElementFoundException(NoSuchElementFoundException exception) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(exception.getMessage());
  }
} 
```

<br>

@ExceptionHandler는 <span style="color:red">**Exception 클래스들을 속성으로 받아 처리할 예외를 지정**</span>할 수 있다. 또한 @ResponseStatus와도 결합가능한데,  만약 ResponseEntity에서도 status를 지정하고 @ResponseStatus도 있다면 ResponseEntity가 우선순위를 갖는다.<br>

@ExceptionHandler를 사용 시에 주의할 점은 <span style="color:red">**@ExceptionHandler에 등록된 예외 클래스와 파라미터로 받는 예와 클래스가 동일**</span>해야 한다는 것이다. 만약 값이 다르다면 스프링은 컴파일 시점에 에러를 내지 않다가 <span style="color:red">**런타임 시점에 에러를 발생**</span>시킨다.<br>

@ExceptionHandler는 컨트롤러에 구현하므로 특정 컨트롤러에서만 발생하는 예외만 처리된다. 하지만 컨트롤러에 에러 처리 코드가 섞이며, <span style="color:red">**에러 처리 코드가 중복**</span>될 가능성이 높다. 그래서 스프링은 전역적으로 예외를 처리할 수 있는 좋은 기술을 제공해준다.

<br><br>

### **@ControllerAdvice와 @RestControllerAdvice**

Spring은 <span style="color:red">**전역적으로 @ExceptionHandler를 적용**</span>할 수 있는 `@ControllerAdvice`와 `@RestControllerAdvice` 어노테이션을 제공한다. 두 개의 차이는 `@Controller`와 `@RestController`와 같이 `@ResponseBody`가 붙어 있어 응답을 Json으로 내려준다는 차이가 있다.<br>

``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {
    ...
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {
    ...
}
```

<br>

ControllerAdvice는 <span style="color:red">**여러 컨트롤러에 대해 전역적으로 ExceptionHandler를 적용**</span>해준다. ControllerAdvice 어노테이션에는 `@Component` 어노테이션이 있어서 ControllerAdvice가 선언된 클래스는 스프링 빈으로 등록된다.<br>

다음과 같이 전역적으로 에러를 핸들링하는 클래스를 만들어 어노테이션을 붙여주면 에러 처리를 위임할 수 있다.<br>

``` java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NoSuchElementFoundException.class)
    protected ResponseEntity<?> handleNoSuchElementFoundException(NoSuchElementFoundException e) {
        final ErrorResponse errorResponse = ErrorResponse.builder()
                .code("Item Not Found")
                .message(e.getMessage()).build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
}
```

<br>

 ControllerAdvice를 이용함으로써 다음과 같은 이점을 누릴 수 있다.<br>

 * 하나의 클래스로 모든 컨트롤러에 대해 전역적으로 예외 처리가 가능하다.
 * 직접 정의한 에러 응답을 일관성있게 클라이언트에게 전달할 수 있다.
 * 별도의 try-catch문이 없어 코드의 가독성이 높아진다.

<br><br>

## **Spring의 예외 처리 흐름**
 <hr />

다음과 같은 예외 처리기들은 스프링의 빈으로 등록되어 있고, 예외가 발생하면 순차적으로 다음의 Resolver들이 처리가 가능한지 판별한 후에 예외가 처리된다.<br>

1. **ExceptionHandlerExceptionResolver** : 에러 응답을 위한 Controller나 ControllerAdvice에 있는 ExceptionHandler를 처리한다.<br>
   1. 예외가 발생한 컨트롤러 안에 적합한 @ExceptionHandler가 있는지 검사한다.
   2. 컨트롤러의 @ExceptionHandler에서 처리 가능하다면 처리하고, 그렇지 않으면 ControllerAdvice로 넘어간다.
   3. ControllerAdvice에 적합한 @ExceptionHandler가 있는지 검사하고 없으면 다음 처리기로 넘어간다.
2. **ResponseStatusExceptionResolver** : HTTP 상태 코드를 지정하는 @ResponseStatus 또는 ResponseStatusException을 처리한다.<br>
   1. @ResponseStatus가 있는지 또는 ResponseStatusException인지 검사한다.
   2. 맞으면 ServletResponse의 sendError()로 예외를 서블릿까지 전달되고, 서블릿이 BasicErroController로 요청을 전달한다.
3. **DefaultHandlerExceptionResolver** : 스프링 내부의 기본 예외들을 처리한다.<br>
   1. Spring의 내부 예외인지 검사하여 맞으면 에러를 처리하고 아니면 넘어간다.
4. 적합한 ExceptionResolver가 없으므로 예외가 서블릿까지 전달되고, 서블릿은 Spring Boot가 진행한 자동 설정에 맞게 BasicErrorController로 요청을 다시 전달한다.

<br>

앞서 살펴보았듯 Spring은 BasicErrorController를 구현해두었다. ExceptionHandler나 ControllerAdvice처럼 직접 에러를 반환하는 경우에는 BasicErrorController를 거치지 않지만 @ResponseStatus, ResponseStatusException 등과 같이 직접 에러 응답을 반환하지 않는 경우에는 최종적으로 BasicController를 거쳐 에러가 처리된다. 내부에서는 2번 컨트롤러로 요청이 전달되는 과정이 진행된다.<br>

<hr/>
참고자료<br>
<a href="https://mangkyu.tistory.com/204">https://mangkyu.tistory.com/204</a><br>