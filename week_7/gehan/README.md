# 스프링 스터디 7주차

---

### 1. 서블릿에서의 예외처리

: 만약 서버 내에서 예외사항을 잡지 못하고 WAS(Web Application Server)로 까지 전달되면 tomcat이 기본으로 제공하는 에러 페이지를 볼 수 있다. 실제 웹 서비스에서는 보다 의미 있는 메시지나 페이지를 유저에게 보여줘야 하기 때문에 스프링에서 제공하는 `WebServerFactoryCustomizer` 를 활용해 에러 발생시 에러페이지를 유저에게 보여줄 수 있다.

<aside>
💡 **[오류 페이지 작동 원리]**

1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-
page/500) -> View

</aside>

→ 오류가 발생하면 오류 페이지를 출력하기 위해 WAS 내부에서 다시 한번 호출이 발생한다. 그런데 로그인 인증 체크 같은 경우를 생각해보면, 이미 한번 필터나, 인터셉터에서 로그인 체크를 완료했다. 따라서 서버 내부에서 오류 페이지를 호출한다고 해서 해당 필터나 인터셉트가 한번 더 호출되는 것은 매우 비효율적이다.

### 2. **DispatcherType**

<aside>
💡 **[DispatcherType] 종류**

froward : 다른 서블릿, JSP 호출 시

include : 다른 서블릿, JSP 결과 포함 시

request : 클라이언트 요청

async : 서블릿 비동기 호출

error : 오류 요청

</aside>

`filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST)` 

: 오류 발생시 비효율적으로 필터와 인터셉터를 거치는 것을 방지하기 위해 필터가 적용되는 범위를 클라이언트 요청으로 한정한다. 생략시 기본 값인 Request에만 적용이 된다. 인터셉터 역시 excludePathPatterns를 활용해 오류 경로 페이지를 제외해주면 불필요한 인터셉터 호출이 사라진다.

### 3. BasicErrorController

: 스프링에서 기본으로 제공하는 기본 오류 페이지 컨트롤러