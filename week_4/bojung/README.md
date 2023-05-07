# week 4 - 스프링 MVC

---

jar를 사용하는 이유?

- 내장 톰캣서버로 최적화하여 사용하고 싶을 때 jar를 사용.

### Log의 기본 사용법

**사용하는 라이브러리**

- SLF4J - 로그 라이브러리를 통합하여 인터페이스로 제공 (롬복이 제공
- Logback - 인터페이스를 구현한 로그 라이브러리

**코드**

`private final Logger log = LoggerFactory.GetLogger(getClass())`

`log.info("log ={}", 변수명)`;

**출력 포멧**

`2023-03-16T20:32:14.916+09:00  INFO 1028 --- [nio-8080-exec-3] h.springmvc.basic.LogTestController : info log=Spring`

시간 / 로그 레벨 / 프로세스 ID / 쓰레드 명 / 클래스명 / 로그 메시지

**********************로그 레벨**********************

*TRACE > DEBUF > INFO > WARN > ERROR*

- 개발 서버는 debug 출력
- 운영 서버는 info 출력

**매핑 정보**

@RestController

- @Controller는 반환 값이 문자열이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링된다.
- @RestController은 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다.

****************************************로그 사용시 장점****************************************

- 로그 레벨에 따라 개발 서버는 모든 로그, 운영 서버는 출력하지 않는 등 상황에 맞게 조절할 수 있다.
- 일반 Sys.out 대비 성능이 좋다.

의문점 :

로그를 출력하고 싶을 때마다 매번 필요한 클래서 전부 로거를 선언해야하는지? 

- 롬복의 Slf4j 애노테이션을 사용하면 선언할 필요 없이 사용 가능하다.

롬복은 Getter, Setter, Slf4j 등 편리한 기능을 제공하나, 현업에서는 롬복 사용을 지양한다는 말을 언뜻 들었다. 그렇다면 롬복을 쓰지 않을 때 사용할 대체 로그 라이브러리는?? 

---

### 요청 매핑

************************************애노테이션 정리************************************

- @Controller : 반환값이 String이면 뷰 이름으로 인식되어 뷰를 찾고 뷰가 랜더링된다.
- @ResponseBody : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
- @RestController : 반환 값으로 뷰를 찾는 것이 아니라 HTTP 메시지 바디에 바로 입력한다. (Controller + ResponseBody)

- **@RequestMapping(value = ”/mapping-address”)**
    - /mapping address URL 호출이 오면 이 메서드가 실행되도록 매핑한다. 다중 설정 가능.
    - method 속성으로 HTTP 메서드를 지정할 수 있다. (GET, POST, PUT, PATCH, DELETE)
    - 더욱 직관적인 방법으로 @**GetMapping(value)**처럼 축약한 애노테이션을 사용할 수 있다. (GET, POST, PATCH, DELETE)
    - @PathVariable(”변수명”)을 사용하면 변수명과 매칭되는 부분을 편리하게 조회할 수 있다. 변수명과 파라미터 이름이 같으면 생략할 수 있다.
        
        ```java
        //PathVariable Example
        @GetMapping("/mapping/users/{userId}/orders/{orderId}")
          public String mappingPath(@PathVariable String userId, @PathVariable Long
          orderId) {
        		...
        	}
        ```
        
    - params = “key=value” : 특정 파라미터 조건 매핑
    - headers = “key=value” : 특정 헤더 조건 매핑
    - 가장 상위에 @RequestMappping(”상대경로”)로 공통의 주소를 미리 선언해놓을 수 있다. 하위 클래스에는 추가되는 경로에 대해서만 *Mapping(”추가 경로”) 애노테이션을 사용하면 된다.

---

### HTTP 요청 파라미터

1. **조회**

**HttpServletRequest**

기존 서블릿을 이용한 조회방법.

**@RequestHeader**

- HTTP 요청 헤더 값을 컨트롤러 메서드의 파라미터로 전달한다(메서드 파라미터가 String가 아니라면 타입변환을 자동으로 적용한다).
- 만약 헤더가 존재하지 않으면 에러가 발생하며, required 속성을 이용해 필수여부를 설정할 수 있다.
- defaultValue 속성을 이용해 기본 값도 설정 가능하다.

**@CookieValue**

- 특정 쿠키를 조회한다.

**@RequestParam(params)**

- 스프링이 제공하며, 요청 파라미터를 편리하게 사용할 수 있다.
- `public String requestParamV3(@RequestParam String username, @RequestParam int age)`
- `String`, `int`, `integer` 등의 단순 타입이면 애노테이션도 생략 가능
- required = boolean : 파라미터 필수 여부, 기본값 = true
- defaultValue = “value” : 파라미터 기본 값 설정. 빈 문자가 들어온 경우에도 기본 값이 적용된다.

**@ModelAttribute**

- 요청 파라미터를 받아 객체를 만들고 그 객체에 값을 넣어주는 과정을 자동화해주는 애노테이션이다.
- 즉, 객체(단순 타입은 RequestParam으로 처리됨)를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXXX)으로 입력해준다.
- 생략 가능하며, RequestParam과 혼란이 생길 수 있으니 단순 타입 외의 나머지, 예를들어 객체의 경우 ModelAttribute로 인식된다.
- 애노테이션 생략시, 객체의 클래스명 앞글자를 소문자로 하는 이름으로 모델에 데이터가 담긴다.
    
    ex.1 @ModelAttribute(”hello”) Item item → 이름을 hello로 지정. 
    
    ex.2 (생략)Item item → 이름을 item으로 지정.
    

******************************단순 텍스트 - Body 및 Header 직접 조회******************************

- **스프링 MVC는 다음 파라미터를 지원한다.**
    - **InputStream(Reader)**: HTTP 요청 메시지 바디의 내용을 직접 조회
    - **OutputStream(Writer)**: HTTP 응답 메시지의 바디에 직접 결과 출력
    - **HttpEntity**: HTTP header, body 정보를 편리하게 조회

1. **응답**
    - HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 응답 메시지를 전달할 수 있다.
    - **ResponseEntity** 엔티티는 HttpEntity 를 상속 받았는데, HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다. ResponseEntity 는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
    - **@ResponseStatus(HttpStatus.OK)** 애노테이션을 사용하면 응답 코드를 설정할 수 있다.

---

### HTTP Message Converter

<aside>
💡 뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 **HTTP 메시지 컨버터**를 사용하면 편리하다.

</aside>

**스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.**

- HTTP 요청: @RequestBody , HttpEntity(RequestEntity)
- HTTP 응답: @ResponseBody , HttpEntity(ResponseEntity)

- canRead() , canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- read() , write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

****************************기본제공 메시지 컨버터****************************

- 0 = ByteArrayHttpMessageConverter : byte[] 데이터를 처리한다.
    - 클래스 타입: byte[] , 미디어타입: */*

- 1 = StringHttpMessageConverter : String 문자로 데이터를 처리한다.
    - 클래스 타입: String , 미디어타입: */*
    
- 2 = MappingJackson2HttpMessageConverter : application/json
    - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련

그럼 HTTP API 방식(데이터를 http message body 에 text 또는 JSON 형식으로 직접 넣어 전달)은 언제 사용해야하는 걸까?

→ 백엔드 개발자는 HTML 뷰 템플릿을 직접 만지는 대신에, HTTP API를 통해 웹 클라이언트가 필요로 하는 데이터와 기능을 제공하면 된다.

---

## Spring MVC - 웹페이지 만들기

### RPG Post/Redirect/Get

중복 등록 문제 해결

→ 저장, 추가 등의 Post 메소드 요청은 재요청시 문제가 생길 수 있으므로 해당 요청 이후에 다른 화면으로 리다이렉트를 호출해주는 방법.

**@RedirectAttributes**

redirect를 더욱 편리하게 사용할 수 있도록하는 기능들을 제공한다.

- URL인코딩 : `redirect:/basic/items/{itemId}`
- pathVariable 바인딩 : `{itemId}`
- 나머지는 쿼리 파라미터로 처리 : `status=true`





# 스프링 MVC 2편

---

## ********************************메세지, 국제화********************************

**********메시지**********

‘label’에 있는 단어를 변경하려면 다 찾아가면서 모두 변경해야 한다. 화면이 수십개 이상이라면 수심개의 파일을 모두 고쳐야 한다. 이런 다양한 메시지를 한 곳에서 관리하도록 하는 기능을 메시지 기능이라 한다.

**************국제화**************

메시지 파일 'messages.properties를 각 나라별로 별도로 관리하면 서비스를 국제화 할 수 있다.

스프링은 기본적인 메시지와 국제화 기능을 모두 제공한다.

resources 하위에 *.properties 파일을 생성하여 사용한다.
