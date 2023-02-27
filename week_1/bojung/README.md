## 웹 애플리케이션 이해

**웹 서버(Web Server)**

- HTTP 기반으로 동작
- 정적 리소스 제공

**웹 애플리케이션 서버(WAS - Web Application Server)**

- HTTP 기반으로 동작
- 프로그램 코드를 실행해서 애플리케이션 로직 수행
- 애플리케이션 코드를 수행하는데  더 특화

******************************************웹 시스템 구성 - WEB, WAS, DB******************************************

- 웹 서버는 정적 리소스 처리, WAS는 중요한 애플리케이션 로직 처리 전담
- 역할이 나뉘어 특정 서버만 증설하는 등의 효율적인 리소스 관리가 가능
- WAS, DB 장애시, WEB 서버가 오류 화면 제공 가능

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1a25c447-4034-4cef-ad28-e3594bc4502f/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b72ba4a7-5601-4554-96a3-d17f5882378e/Untitled.png)

**HTTP 요청, 응답 흐름 :**

1. 사용자가 URL(또는 IP)을 통해 WEB 서버를 호출하고 요청사항을 객체(request)에 담아 전송
2. WEB 서버는 요청 객체(request)를 받아서 바로 처리하거나 어플리케이션 서버(WAS)로 객체 전달
3. WAS 서버는 요청에 대한 내용과 요청 객체(request)를 받아 적절히 처리 (필요시 DB 작업 진행)
4. WAS 서버는 처리 후 결과를 응답 객체(response)에 담아 WEB서버로 회신
5. WEB 서버는 응답 객체(response)를 다시 사용자에게 회신
6. 사용자의 브라우저는 WEB 서버가 보내준 코드를 해석해 화면을 구성하여 출력

****************서블릿(Servlet) :****************

웹 서버의 성능을 향상하기 위해 사용되는 자바 클래스의 일종이다. HTTP 요청, 응답 정보를 편리하게 제공한다.

****************************************서블릿 컨테이너 :****************************************

- 서블릿을 지원하는 WAS (ex. Tomcat)
- 서블릿 객체의 생명주기 관리
- 서블릿 객체는 싱글톤으로 관리하여 모든 요청은 동일한 서블릿 객체 인스턴스에 접근하므로 **공유변수 사용에 주의**가 필요하다. 서블릿 컨테이너 종료시 함께 종료된다.
- 동시 요청을 위한 멀티 스레드 처리 지원

**동시 요청 - 멀티 스레드**

하나의 요청 당 하나의 스레드가 사용된다. 따라서 다수의 요청을 동시에 처리하기 위해서는 여러 스레드가 필요하다. WAS는 멀티스레드를 지원하며 따라서 개발자가 멀티 스레드 관련 코드를 신경쓰지 않아도 된다.

**스레드 풀 :**

작업 처리에 사용되는 스레드를 제한된 개수만큼 정해놓고, 작업 큐에 들어오는 작업들을 하나씩 맡아 처리하는 스레드 제어 방식. 

- 사용을 종료하면 스레드 풀에 해당 스레드를 반납
- 제한된 수를 넘는 요청이 올 경우 대기, 거절 설정 가능
- 스레드를 생성하고 종료하는 비용이 절약, 빠른 응답 시간
- 요청이 넘쳐도 기존 요청의 안전한 처리 가능
- **Max Thread 튜닝이 매우 중요** (서버 리소스 여유, 응답 지연 ↔ 메모리 리소스 임계점 초과, 서버 다운), 성능 테스트 필요

**********************************************서버 사이드, 클라이언트 사이드 렌더링 :**********************************************

- SSR :
    - HTML 최종 결과를 서버에서 만들어서 웹 브라우저에 전달
    - 주로 정적인 화면에 사용
- CSR :
    - HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성해서 적용
    - 주로 동적인 화면에 사용, 부분 변경 가능

---

## 서블릿

@ServletComponentScan : 

스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 @ServletComponentScan 을 지원한다

@WebServlet(name, urlPattern)

HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 Service(HttpServletRequest req, HttpServletResponse res) 메서드를 실행한다.

```java
protected void service(HttpServletRequest req, HttpServletResponse res)
	// 요청 정보 조회
	String username = request.getParameter("username");

	// 응답 정보 등록
	response.setContentType("text/plain");
	response.getWriter().write("hello " + username); // 응답 메세지 Body에 데이터를 입
```

**HTTP 요청 데이터**

- **GET - Query Parameter**
    - 주로 검색, 필터, 페이징 등에서 많이 사용하는 방식

```java
//단일 파라미터 조회
String username = request.getParameter("username"); 

//파라미터 이름들 모두 조회
Enumeration<String> parameterNames = request.getParameterNames(); 

//파라미터를 Map으로 조회
Map<String, String[]> parameterMap = request.getParameterMap(); 

//복수 파라미터 조회
String[] usernames = request.getParameterValues("username"); 
```

- **POST - HTML Form**
    - 주로 회원 가입, 상품 주문 등에서 사용하는 방식
    - Content-Type : application/x-www-form-urlencoded
    - 메세지 바디에 쿼리 파라미터 형식으로 데이터를 전달하여 바디 데이터 조회 방식이 쿼리 파라미터 조회 메서드와 호환된다.

- **API 메시지 바디 - plain text**
    - HTTP message body에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용, JSON, XML, TEXT
    - POST, PUT, PATCH 메서드 사용 가능

```java
// Message Body Data
ServletInputStream inputStream = request.getInputStream();
 String messageBody = StreamUtils.copyToString(inputStream,
StandardCharsets.UTF_8);
 System.out.println("messageBody = " + messageBody);

// 출력 결과 :
// postman 에서 입력한 message body 값이 출력된다.
```

- **API 메세지 바디 - JSON**
    - JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson, Gson 같은 JSON 변환
    라이브러리를 추가해서 사용해야 한다.
    - Spring Boot는 기본적으로 jackson을 JSON Library로 제공한다.
    - Jackson 라이브러리의 ObjectMapper 메서드를 이용하면 JSON을 Java 객체로 변환할 수 있고, 반대로 Java 객체를 JSON 객체로 serialization 할 수 있다.
    
    ```java
    import com.fasterxml.jackson.databind.ObjectMapper;
    			...
    private ObjectMapper objectMapper = new ObjectMapper();
    
    	@Override
    	protected void service(...) {
    
    	ServletInputStream inputStream = request.getInputStream();
    	String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    	//JSON 객체를 Java 객체로 변환
    	HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
    
    	System.out.println("helloData.username = " + helloData.getUsername());
    	System.out.println("helloData.age = " + helloData.getAge());
    ```
    

**HTTP 응답 데이터** 

역할 :

- HTTP 응답 코드 지정
- 헤더 생성
- 바디 생성
- Content-Type, Cookie, Redirect

```java
//[status-line]
response.setStatus(HttpServletResponse.SC_OK); //응답코드 지정

//[response-headers]
response.setHeader("Content-Type", "text/plain;charset=utf-8"); //헤더 정보 설정

//[message body]
PrintWriter writer = response.getWriter();
writer.println("ok");

//[Header 편의 메서드]

//Content-Type: text/plain;charset=utf-8
response.setContentType("text/plain");
response.setCharacterEncoding("utf-8");

//Set-Cookie: myCookie=good; Max-Age=600;
Cookie cookie = new Cookie("myCookie", "good");
cookie.setMaxAge(600); //600초
response.addCookie(cookie);

//Location: /basic/hello-form.html
response.sendRedirect("/basic/hello-form.html");
```

**응답 데이터 - Plain text, HTML**

- HTTP 응답으로 HTML을 반환할 때는 content-type을 text/html 로 지정해야 한다.
- HTML 형식을 전부 println으로 출력해야 하므로 매우 번거롭다.

**응답 데이터 - API JSON**

- HTTP 응답으로 JSON을 반환할 때는 content-type을 application/json 로 지정해야 한다.
- Jackson 라이브러리가 제공하는 objectMapper.writeValueAsString() 를 사용하면 객체를 JSON
문자로 변경할 수 있다.

```java
	...
private ObjectMapper objectMapper = new ObjectMapper();

@Override
protected void service(...) ... {

//Content-Type: application/json
response.setHeader("content-type", "application/json");
response.setCharacterEncoding("utf-8");

HelloData data = new HelloData();
data.setUsername("kim");
data.setAge(20);

//{"username":"kim","age":20}
String result = objectMapper.writeValueAsString(data);
response.getWriter().write(result);
```

- Tip
    - [main/resources/application.properties] 에 `logging.level.org.apache.coyote.http11=debug` 를 입력하면 상세 요청 정보를 콘솔창에서 확인할 수 있다.

---

## 서블릿, JSP, MVC 패턴

******************************서블릿으로 웹 애플리케이션 만들기******************************

서블릿과 자바 코드만으로 웹 애플리케이션을 만들면..

- 서블릿 덕분에 동적으로 원하는 HTML을 생성할 수 있지만, 코드가 복잡하고 비효율적이다.
- **템플릿 엔진**은 이런 서블릿의 단점을 보완하기 위해 만들어졌으며, HTML 문서에서 필요한 곳만 코드를 적용해서 동적으로 변경할 수 있다.
- 템플릿 엔진에는 JSP, Thymeleaf, Freemarker, Velocity등이 있다.

**JSP로 웹 애플리케이션 만들기**

`<%@ page contentType="text/html;charset=UTF-8" language="java" %>`

- .jsp 문서의 첫 줄은 JSP 문서라는 뜻이다.
- 자바 코드는 <% ~~~ %> 구문 안에 위치시킨다.
- 자바 코드 출력은 <%= ~~ %> 구문 안에 위치시킨다.
- JSP는 Servlet 파일(.java)로 변환된다. 변환된 서블릿 파일을 다시 컴파일해서 .class 파일로 만든 뒤 실행하며 실행 결과는 자바 언어가 모두 사라진 HTML 코드가 된다.
    - 따라서 request, response 객체를 따로 선언하지 않고 사용 가능하다.
    - + 처음 구동할 때는 파일 변환 과정이 한 번 더 있으므로 서블릿보다 느리지만, 첫 구동에서 class 파일을 생성해두면 두 번째부터는 변환과정 및 컴파일 과정이 없어 서블릿과 거의 동일하게 작동한다.

**서블릿과 JSP의 한계**

- JAVA 코드, 데이터를 조회하는 리포지토리 등등 다양한 코드가 모두 JSP에 노출되어 있다. JSP가 너무 많은 역할을 한다.
- 이 문제를 해결하기 위해 비즈니스 로직은 서블릿 처럼 다른곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면(View)을 그리는 일에 집중하도록 하는 MVC 패턴이 등장했다.
