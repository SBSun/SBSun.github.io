---
title: "[Spring] @RestControllerAdvice를 이용한 Spring 예외 처리 방법"
excerpt: "@RestControllerAdvice를 이용한 예외 처리 방법에 대해서 알아보자"

categories:
  - Spring
tags:
  - [Spring]

published: true

permalink: /spring/restcontrolleradvice-exception-handling/

toc: true
toc_sticky: true

date: 2023-03-23
last_modified_at: 2023-03-23

--- 

## **ControllerAdvice란**
<hr />

ControllerAdvice는 <span style="color:red">**여러 컨트롤러에 대해 전역적으로 ExceptionHandler를 적용**</span>해준다. `@ControllerAdvice` 어노테이션에는 `@Component` 어노테이션이 포함되어 있어서 스프링 빈으로 등록된다. 그러므로 다음과 같이 전역적으로 에러를 핸들링하는 클래스를 만들어 어노테이션을 붙여줌으로써 에러 처리를 위임할 수 있다.<br>

``` java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(RestApiException.class)
    public ResponseEntity<Object> handleCustomException(RestApiException e) {
        ErrorCode errorCode = e.getErrorCode();
        return handleExceptionInternal(errorCode);
    }
}
```

<br>

**ControllerAdvice의 이점**
* 하나의 클래스로 모든 컨트롤러의 대해 전역적으로 예외 처리가 가능하다.
* 직접 정의한 에러 응답을 일관성있게 클라이언트에게 전달할 수 있다.
* 별도의 try-catch문이 없어 코드의 가독성이 높아진다.

<br>

**ControllerAdvice를 사용할 때 주의점**
* 한 프로젝트당 하나의 ControllerAdvice만 관리하는 것이 좋다.
* 만약 여러 ControllerAdvice가 필요하다면 basePackage나 annotations 등을 지정해야 한다.
* 직접 구현한 ExceptionHandler 클래스들은 한 공간에서 관리한다.

<br><br>

## **@RestControllerAdvice를 이용한 Spring 예외 처리 방법**
<hr />

### **에러 코드 정의**

먼저 클라이언트에게 보내줄 에러 코드를 정의해야 한다. 
<br>기본적으로 에러 이름과 HTTP 상태 및 메세지를 가지고 있다.

에러 코드는 애플리케이션에서 전역적으로 사용되는 **CommonErrorCode**와 특정 도메인에 대해 구체적으로 내려가는 **UserErrorCode**로 나누고, 인터페이스를 이용해 추상화한다.<br>

``` java
public interface ErrorCode {

    String name();
    HttpStatus getHttpStatus();
    String getMessage();
}
```

<br>

그리고 발생할 수 있는 에러 코드를 정의한다.<br>

``` java
@Getter
@RequiredArgsConstructor
public enum CommonErrorCode implements ErrorCode{

    // 400 BAD_REQUEST 잘못된 요청
    INVALID_PARAMETER(HttpStatus.BAD_REQUEST, "파라미터 값을 확인해주세요."),

    // 404 NOT_FOUND 잘못된 리소스 접근
    RESOURCE_NOT_FOUND(HttpStatus.NOT_FOUND, "존재하지 않는 값입니다."),
    CATEGORY_NOT_FOUND(HttpStatus.NOT_FOUND, "존재하지 않는 카테고리입니다."),
    PARENT_CATEGORY_NOT_FOUND(HttpStatus.NOT_FOUND, "존재하지 않는 상위 카테고리입니다."),

    // 409 CONFLICT 중복된 리소스
    ALREADY_SAVED_CATEGORY(HttpStatus.CONFLICT, "이미 존재하는 카테고리입니다."),

    // 500 INTERNAL SERVER ERROR
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "서버 에러입니다. 서버 팀에 연락주세요!");

    private final HttpStatus httpStatus;
    private final String message;
}

@Getter
@RequiredArgsConstructor
public enum UserErrorCode implements ErrorCode{

    // 403 FORBIDDEN 권한 없음
    INACTIVE_USER(HttpStatus.FORBIDDEN, "You don't have permission to access");

    private final HttpStatus httpStatus;
    private final String message;
}
```

<br><br>

### **예외 클래스 생성**

발생할 예외를 처리해줄 예외 클래스(Exception Class)를 추가해주어야 한다.<br> <span style="color:red">**언체크 예외(런타임 예외)를 상속받는 예외 클래스**</span>를 추가하자.

``` java
@Getter
@RequiredArgsConstructor
public class RestApiException extends RuntimeException {

    private final ErrorCode errorCode;
}
```

<br>

언체크 예외를 상속받은 이유는 일반적인 비지니스 로직들은 따로 catch해서 처리할 것이 없으므로 만약 체크 예외로 한다면 불필요하게 throws가 전파될 것이기 때문이다.<br>

또한 Spring은 <span style="color:red">**내부적으로 발생한 예외를 확인하여 언체크 예외이거나 에러라면 자동으로 롤백**</span>시키도록 처리한다.

<br><br>

### **에러 응답 클래스 생성**

에러 응답 클래스를 생성하여 클라이언트에게 일관된 포멧의 에러를 보내주도록 했다.<br>

``` java
@Getter
@Builder
@RequiredArgsConstructor
public class ErrorResponse {

    private final String code;
    private final String message;

    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    private final List<ValidationError> errors;

    @Getter
    @Builder
    @RequiredArgsConstructor
    public static class ValidationError {
    
        private final String field;
        private final String message;

        public static ValidationError of(final FieldError fieldError) {
            return ValidationError.builder()
                    .field(fieldError.getField())
                    .message(fieldError.getDefaultMessage())
                    .build();
        }
    }
}
```

<br>

**ValidationError** 정적 클래스는 `@Vaild`를 사용했을 때 유효성 검사에서 에러가 발생했을 때 어느 필드에서 에러가 발생했는지 응답을 위한 클래스이다. 만약 errors가 없다면 응답으로 내려가지 않도록 `@JsonInclude` 어노테이션을 추가했다.

<br><br>

### **@RestControllerAdvice 구현**

이제 <span style="color:red">**전역적으로 에러를 처리**</span>해주는 `@RestControllerAdvice` 클래스를 구현하자.<br>

Spring은 <span style="color:red">**스프링 예외를 미리 처리해둔 ResponseEntityExceptionHandler를 추성 클래스로 제공**</span>하고 있다. ResponseEntityExceptionHandler에는 ExceptionHandle가 모두 구현되어 있으므로 <span style="color:red">**ControllerAdvice 클래스가 이를 상속**</span>받게 하면 된다. 하지만 에러 메세지는 반환하지 않으므로 스프링 예외에 대한 에러 응답을 보내려면 아래 메서드를 오버라이딩 해야 한다.<br>

``` java
public abstract class ResponseEntityExceptionHandler {
    ...

    protected ResponseEntity<Object> handleExceptionInternal(
        Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatus status, WebRequest request){
            
        ...
    }
}
```

<br>

RestApiException 예외와 `@Vaild`와 `@Vaildated`에 의한 유효성 검증에 실패했을 때 반환하는 MethodArgumentNotValidException 예외와 ConstraintViolationException 예외, 마지막으로 잘못된 파라미터를 넘겼을 경우 발생하는 IllegalArgumentException 에러를 처리한다.<br>

``` java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(RestApiException.class)
    public ResponseEntity<Object> handleCustomException(RestApiException e) {
        ErrorCode errorCode = e.getErrorCode();
        return handleExceptionInternal(errorCode);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Object> handleIllegalArgument(IllegalArgumentException e) {
        log.warn("handleIllegalArgument", e);
        ErrorCode errorCode = CommonErrorCode.INVALID_PARAMETER;
        return handleExceptionInternal(errorCode, e.getMessage());
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<Object> handleConstraintViolation(ConstraintViolationException e){
        ErrorCode errorCode = CommonErrorCode.INVALID_PARAMETER;
        return handleExceptionInternal(errorCode);
    }

    @Override
    public ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException e,
            HttpHeaders headers,
            HttpStatus status,
            WebRequest request) {
        log.warn("handleIllegalArgument", e);
        ErrorCode errorCode = CommonErrorCode.INVALID_PARAMETER;
        return handleExceptionInternal(e, errorCode);
    }

    private ResponseEntity<Object> handleExceptionInternal(ErrorCode errorCode) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(errorCode));
    }

    private ErrorResponse makeErrorResponse(ErrorCode errorCode) {
        return ErrorResponse.builder()
                .code(errorCode.name())
                .message(errorCode.getMessage())
                .build();
    }

    private ResponseEntity<Object> handleExceptionInternal(ErrorCode errorCode, String message) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(errorCode, message));
    }

    private ErrorResponse makeErrorResponse(ErrorCode errorCode, String message) {
        return ErrorResponse.builder()
                .code(errorCode.name())
                .message(message)
                .build();
    }

    private ResponseEntity<Object> handleExceptionInternal(BindException e, ErrorCode errorCode) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(e, errorCode));
    }

    private ErrorResponse makeErrorResponse(BindException e, ErrorCode errorCode) {
        List<ErrorResponse.ValidationError> validationErrorList = e.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(ErrorResponse.ValidationError::of)
                .collect(Collectors.toList());

        return ErrorResponse.builder()
                .code(errorCode.name())
                .message(errorCode.getMessage())
                .errors(validationErrorList)
                .build();
    }
}
```

<br>

RestApiException과 IllegalArgumentException의 경우에는 이를 캐치해서 핸들링하는 `@ExceptionHandler`를 구현해주면 된다. 하지만 `@Vaild`에 의한 MethodArgumentNotValidException의 경우에는 에러 필드와 메세지를 추가해주어야 하는데, 관련 정보는 MethodArgumentNotValidException의 getBindingResult을 통해서 얻을 수 있다.

<br><br>

### **에러 응답 확인**

``` java
@Validated
@RestController
@RequiredArgsConstructor
public class UserController {

    @GetMapping("/user/checkEmail")
    public boolean checkEmail(@RequestParam @NotBlank @Email String email) {

        return userService.checkEmail(email);
    }
}
```

<br>

해당 API의 파라미터 값으로 공백을 넣어준다면 원하는 대로 에러 응답이 보내진다.<br>

``` json
{
    "code": "INVALID_PARAMETER",
    "message": "checkEmail.email: 공백일 수 없습니다"
}
```

<hr />
참고자료<br>
<a href="https://mangkyu.tistory.com/205">https://mangkyu.tistory.com/205</a><br>