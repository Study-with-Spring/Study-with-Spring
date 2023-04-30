Java의 메인쓰레드에서 예외가 발생하면 예외 정보를 남기고 메인쓰레드는 종료된다.
하지만 웹 어플리케이션에서 예외가 발생했는데 try-catch로 잡지 않으면 WAS까지 넘어온다면 어떻게 될까?
400번대나 500번대 에러를 볼 수 있다.

# 서블릿 예외 처리 - 오류 페이지 동작 원리

서블릿에서 Exception이 발생하여 서블릿 외부로 보내지거나 response.error() 가 호출되었을 때, 설정된 오류 페이지를 찾습니다.
이 때 sendError() 흐름이 `컨트롤러 -> 인터셉터 -> 필터 -> 서블릿 -> WAS` 순으로 호출됩니다. 그리고 오류페이지 요청 역시 역순으로 호출이 되겠죠. WAS는 오류 페이지 요청과 동시에 오류정보를 request의 attribute에 추가해서 넘겨줍니다. <br>

근데 이 때 필터와 인터셉트는 처음에 로그인 인증을 마쳤기 때문에 재호출 될 필요는 없습니다. 클라이언트로부터 발생한 요청과 오류페이지 내부 요청을 구분할 수 있다면 효율적으로 필터와 인터셉트를 거칠 수 있겠죠. <br>

**DispatcherType.ERROR**

그래서 dispatcherTypes 라는 옵션을 사용합니다. 인터셉트의 경우 서블릿이 아닌 스프링에서 제공하는 기능이라 DispatcherType 과 무관하게 항상 호출됩니다. excludesPathPatterns() 를 사용해서 오류 페이지 요청을 제외시킬 수 있습니다. <br>

```java
 @Override
 public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
        .order(1)
        .addPathPatterns("/**")
        .excludePathPatterns(
        "/css/**", "/*.ico"
        , "/error", "/error-page/**" //오류 페이지 경로
    );
 }
```

## BasicErrorController

사실 BasicErrorController 라는 스프링 컨트롤러에 오류페이지 기능이 다 있어요. <br>
개발자가 할 일은 그저 resources/templates/error/ 에 '404.html' 처럼 뷰파일을 만들기만 하면 됩니다. <br>
BasicErrorController 는 다음 정보를 바탕으로 뷰에 전달하는데 뷰 템플릿에서 이를 활용할 수 있습니다.

```java
* timestamp: Fri Feb 05 00:00:00 KST 2021
* status: 400
* error: Bad Request
* exception: org.springframework.validation.BindException
* trace: 예외 trace
* message: Validation failed for object='data'. Error count: 1
* errors: Errors(BindingResult)
* path: 클라이언트 요청 경로 (`/hello`)
```

오류 정보 추가 => `resources/templates/error/500.html`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
</head>
<body>
<div class="container" style="max-width: 600px">
    <div class="py-5 text-center">
        <h2>500 오류 화면 스프링 부트 제공</h2>
    </div>
    <div>
        <p>오류 화면 입니다.</p>
    </div>
    <ul>
        <li>오류 정보</li>
        <ul>
            <li th:text="|timestamp: ${timestamp}|"></li>
            <li th:text="|path: ${path}|"></li>
            <li th:text="|status: ${status}|"></li>
            <li th:text="|message: ${message}|"></li>
            <li th:text="|error: ${error}|"></li>
            <li th:text="|exception: ${exception}|"></li>
            <li th:text="|errors: ${errors}|"></li>
            <li th:text="|trace: ${trace}|"></li>
        </ul>
        </li>
    </ul>
    <hr class="my-4">
</div> <!-- /container -->
</body>
</html>
```

오류 관련 정보를 고객에게 숨길 수도 있습니다.
`application.properties`

    ```
    server.error.include-exception=false #예외 정보
    server.error.include-message=never #오류 메시지
    server.error.include-binding-errors=never #validation 오류
    server.error.include-stacktrace=never #예외 trace
    ```

옵션은 다음 3 가지가 있습니다.

- never: 오류 정보 미표시
- always: 오류 정보 표시
- on-param: 요청 파라미터에 debug 라는 값이 있을 때만 오류 정보 표시
