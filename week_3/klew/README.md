# MVC 패턴 흐름

**Dispatcher Servlet**

디스패처 서블릿이란 서블릿 컨테이너의 가장 앞단에서 HTTP로 들어오는 모든 요청을 먼저 받아 매핑된 컨트롤러에 배차해주는 "프론트 컨트롤러"입니다.<br>
Dispatcher Servlet은 모든 요청에 대해 doDispatch()를 최종적으로 호출합니다.

- 프론트 컨트롤러 (Front Controller) ??

컨트롤러 호출전에 "공통되는 기능"들을 처리하기 위한 입구 컨트롤러입니다. 프론트 컨트롤러 도입 전에는 아래와 같이 각각의 컨트롤러마다 공통되는 기능과 추가적인 기능을 모두 작성하는 문제가 있었지만 프론트 컨트롤러에서 공통 기능을 수행한 뒤, 각각의 요청에 대응되는 컨트롤러를 호출함으로써 공통 기능의 처리가 가능해졌습니다.

**doDispatch 동작 순서**

> 1. HandlerMapping들을 통해서 요청 URL에 매핑된 핸들러(컨트롤러)를 조회합니다.
> 2. 조회된 핸들러를 실행할 수 있는 핸들러 어댑터를 조회합니다.
> 3. 핸들러 어댑터가 실제 핸들러를 실행합니다. 이때 핸들러 어댑터는 HandlerAdapter 인터페이스를 구현해서 생성합니다.
> 4. 핸들러가 ModelAndView를 반환합니다.
> 5. ModelAndView를 알맞은 View로 전달하기 위해 DispatcherServlet에 의해 "뷰 리졸버" 가 호출합니다.

- View Resolver (뷰 리졸버) ??

controller가 Dispatcher Servlet에게 ModelAndView(data, view name, ...) 객체를 보내면 Dispatcher Servlet에서 객체 정보를 추출하여 URL 경로를 만듭니다. 뷰 리졸버는 ModelAndView 객체를 View 영역으로 전달하기 위해 알맞은 View 정보를 설정하는 역할을 한다.
