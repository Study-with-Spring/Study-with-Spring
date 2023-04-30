# week 8 - 예외처리

## **오류 페이지**

**순수 컨테이너로의 예외처리**

1. **Exception**
    
    오류를 애플리케이션에서 잡지 못하고, 서블릿 밖으로 예외가 전달되면 tomcat이 기본으로 제공하는 404 오류 화면을 띄운다.
    
    스프링에서도 기본 오류 화면을 제공한다.
    

1. **response.sendError**
    
    HttpServletResponse 가 제공하는 sendError 라는 메서드를 사용할 수 있다.
    
    response.sendError()를 호출하면 response 내부에는 오류가 발생했다는 상태를 저장해둔다. 그리고 서블릿 컨테이너는 고객에게 응답 전에 response 에 sendError() 가 호출되었는지 확인한다. 호출되었다면 설정한 오류 코드에 맞추어 기본 오류 페이지를 보여준다.
    

### **실제 사용**

**서블릿 예외 처리**

```java
ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");
```

sendError 흐름 : WAS ← 필터 ← 서블릿 ← 인터셉터 ← 컨트롤러

WAS는 오류페이지를 출력하기 위해 지정된 페이지를 다시 요청하고 오류 정보를 request 의 attribute 에 추가해서 넘겨준다.

DispatcherType : 중복 필터 호출을 제한하기 위해 클라이언트로부터의 요청인지, 오류 페이지를 출력하기 위한 내부 요청인지 구분하기 위한 추가 정보. (FORWARD, INCLUDE, REQUEST, AYSNC, ERROR)

excludePathPatterns : 중복 인터셉터 호출을 제한할 수 있다.

### 스프링 부트 오류 페이지

스프링은 위 과정을 기본으로 제공한다.

기본 오류 페이지 ErrorPage와 BasicErrorController라는 스프링 컨트롤러를 자동으로 등록한다. 개발자는 오류 페이지 화면만 BasicErrorController 가 제공하는 룰과 우선순위에 따라서 등록하면 된다.

---

## API 예외 처리

오류 페이지는 단순히 고객에게 오류 화면을 보여주고 끝이지만, API는 각 오류 상황에 맞는 오류 응답 스펙을 정하고, JSON으로 데이터를 내려주어야 한다.

마찬가지로 스프링 부트가 제공하는 기본 오류 방식을 사용할 수 있다.

BasicErrorController

errorHtml() : 클라이언트 요청의 Accept 헤더 값이 text/html인 경우 호출해서 view를 제공한다.

error() : 그 외 경우에 호출되고 HTTP body에 JSON 데이터를 반환한다.

**HandlerExceptionResolver**

컨트롤러 밖으로 예외가 던져진 경우 예외를 해결하고 동작을 새로 정의할 수 있는 기능. 따라서 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않아 스프링 MVC에서 모두 예외를 모두 처리할 수 있다는 것이 핵심이다.

반환 값에 따라 DispatcherServlet 동작 방식이 다르다.

**빈 ModelAndView**: new ModelAndView() 처럼 빈 ModelAndView 를 반환하면 뷰를 렌더링 하지 않고, 정상 흐름으로 서블릿이 리턴된다.

**ModelAndView 지정**: ModelAndView 에 View , Model 등의 정보를 지정해서 반환하면 뷰를 렌더링 한다.

**null**: null 을 반환하면, 다음 ExceptionResolver 를 찾아서 실행한다. 만약 처리할 수 있는 ExceptionResolver 가 없으면 예외 처리가 안되고, 기존에 발생한 예외를 서블릿 밖으로 던진다.

### ExceptionHandlerExceptionResolver

스프링은 에러 처리라는 공통 관심사(cross-cutting concerns)를 메인 로직으로부터 분리하는 예외 처리 방식을 제공한다.

ExceptionHandlerExceptionResolver 를 기본으로 제공하고, 기본으로 제공하는 ExceptionResolver 중에 우선순위도 가장 높다. 실무에서 API 예외 처리는 대부분 이 기능을 사용한다.

@ExceptionHandler 와 @ControllerAdvice 를 조합하면 예외를 깔끔하게 해결할 수 있다.

**@ExceptionHandler**

애노테이션을 선언하고 해당 컨트롤러에서 처리하고 싶은 예외를 지정해준다. 해당 컨트롤러에서 예외가 발생하면 이 메서드가 호출된다.

**@ControllerAdvice**

컨트롤러마다 중복된 코드를 사용하는 문제를 해결해준다. 

- 대상으로 지정한 여러 컨트롤러에 @ExceptionHandler , @InitBinder 기능을 부여해주는 역할을 한다.
- 대상을 지정하지 않으면 모든 컨트롤러에 적용된다.
