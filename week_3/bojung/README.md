프론트 컨트롤러 :

각 클라이언트들은 Front Controller에 요청을 보내고, **Front Controller은 각 요청에 맞는 컨트롤러를 찾아서 호출**시킨다.

즉, **공통 코드**에 대해서는 **Front Controller에서 처리**하고, 서로 다른 코드들만 각 Controller에서 처리할 수 있도록 한다.

**DispatcherServlet (프론트 컨트롤러 역할)**

- 서블릿으로 동작
- 모든 경로에 대해 매핑

DispacherServlet.doDispatch() 동작 순서:

핸들러 조회 → 핸들러 어댑터 조회 → 핸들러 어댑터 실행 → 핸들러 실행 → ModelAndView 반환 → viewResolver 호출 → View 반환 → 뷰 렌더링

## 스프링 부트가 자주 사용하는 핸들러 매핑, 어댑터

**@RequestMapping (99% 사용)**

**HandlerMapping :**

RequestMappingHandlerMapping : 애노테이션 기반

********************************HandlerAdapter :********************************

RequestMappingHandlerAdapter : 애노테이션 기반

**********************뷰 리졸버 :**********************

- url의 중복되는 앞단과 뒷단을 Dispatcher Servlet에서 처리하도록 한다.
- InternalResourceViewResolver 라는 뷰 리졸버를 자동으로 등록하는데, 이때
application.properties 에 등록한 spring.mvc.view.prefix , spring.mvc.view.suffix 설정
정보를 사용해서 등록한다.
