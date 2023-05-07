# 스프링 스터디 6주차

---

: 이번 강의는 웹서비스의 기본이자 가장 중요한 로그인 구현에 대해 알아본다.

### 1. 도메인이란?

> “web은 domain을 알고 있지만 domain은 web을 모르도록 설계해야 한다.”
> 

: 시스템이 구현해야 하는 핵심 비즈니스 업무 영역을 말함. 따라서 WEB 기술이 바뀐다 하더라도 도메인은 그대로 유지되어야 한다. 따라서 domain은 web으로 부터 의존관계가 독립적이어야 한다.

### 2. 로그인 구현하기 - 단순 비교

```java
return memberRepository.findByLoginId(loginId)
                  .filter(m -> m.getPassword().equals(password))
                  .orElse(null);
```

: 로그인 구현 첫번째 단계는 입력받은 user의 password가 저장소(db)에 있는 내용과 일치하는지 확인하는 것이다. 위 코드처럼 단순 비교를 통해 동작한다. 하지만 이 경우 **로그인 상태를 유지**하는 것이 불가능하다.

### 3. 로그인 처리하기 - 쿠키 사용

![Untitled](%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%A5%E1%84%83%E1%85%B5%206%E1%84%8C%E1%85%AE%E1%84%8E%E1%85%A1%20259a86134a0247a4a789fffaf2e2eabd/Untitled.png)

: 로그인 상태를 유지하는 방법으로 쿠키를 사용할 수 있다. 쿠키는 키-값 형태로 된 데이터 조각으로 사용자 정보등을 저장한다. 최초 로그인 성공시 서버에서 클라이언트로 쿠키를 같이 전달하고, 클라이언트의 브라우저는 해당 쿠키를 저장해두고 해당 사이트의 모든 요청에 대해 쿠키를 같이 전송함으로써 로그인 상태를 유지한다.

- 영속 쿠키 : 만료 날짜까지만 유지
- 세션 쿠키 : 만료 날짜 생략시 브라우저 종료시 까지만 유지

[쿠키 생성 및 response에 추가]

```java
Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
response.addCookie(idCookie);

// 로그인 성공 시 쿠키 생성 -> HttpServletResponse에 담아서 전송
```

[로그인 시 쿠키 내용으로 로그인 여부 확인]

```java
public String homeLogin(@CookieValue(name = "memberId", required = false) 
												Long memberId, Model model)

//@CookieValue를 통해 편리하게 쿠키 조회 가능
```

[로그아웃 구현]

```java
private void expireCookie(HttpServletResponse response, String cookieName) {
      Cookie cookie = new Cookie(cookieName, null);
      cookie.setMaxAge(0);
      response.addCookie(cookie);
}
```

: 로그아웃은 쿠키를 생성하고 종료 날짜를 0으로 지정하여 클라이언트로 전송하면 된다.

### 4. 쿠키와 보안 문제

: 쿠키에는 심각한 보안 문제가 있다. 우선 쿠키값은 임의로 변경될 수 있다. 변경된 값을 바탕으로 다른 user 정보에 접근할 수 있는 위험이 있고, 쿠키 내부의 정보 자체도 해킹이나 하이재킹 등을 통해 유출될 위험이 있다. 이에 대한 해결책으로 제시하는 것이 **세션**이다.

1. 쿠키에는 토큰 값만 전달 → 서버에서 토큰과 id 매핑
2. 토큰은 랜던값으로 생성
3. 토큰의 만료시간을 걸어 토큰이 유출되더라도 악용되지 못하도록 설정

![Untitled](%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%A5%E1%84%83%E1%85%B5%206%E1%84%8C%E1%85%AE%E1%84%8E%E1%85%A1%20259a86134a0247a4a789fffaf2e2eabd/Untitled%201.png)

**[세션 작동 방식]**

1. 로그인 시 세션 ID 생성 (토큰 값 - UUID, userId) 
2. 쿠키에는 UUID로 된 세션 ID만 전달 (회원과 관련된 정보는 일체 전달하지 않음)
3. 클라이언트는 요청간 토큰이 저장된 쿠키를 전달
4. 서버는 해당 토큰을 조회해 로그인 상태를 유지함.

### 5. 서블릿 HTTP 세션

```java
//세션이 있으면 있는 세션 반환, 없으면 신규 세션 생성
HttpSession session = request.getSession();
//세션에 로그인 회원 정보 보관
session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

//로그아웃 시 세션 삭제
HttpSession session = request.getSession(false);
if (session != null) {
          session.invalidate();
      }
```

: 이것보다 더 편리한 @SessionAttribute 어노테이션

```java
@SessionAttribute(name = "loginMember", required = false) Member loginMember

//아래와 같이 사용된다.
public String homeLoginV3Spring(
          @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false)
  Member loginMember,
          Model model)
```

### 6. 세션 타임아웃

: http는 비연결성이므로 클라이언트가 브라우저를 종료했는지 여부를 알 수 없다. 그렇기 때문에 세션을 삭제해야하는 시점이 언제인지에 대한 의문점이 생길 수 있다. 만약 세션 유지 시간을 30분으로 정해놓는다고 한다면 사용자는 사이트를 사용하다가도 최초 로그인 시점으로부터 30분 뒤면 다시 로그인을 해야하는 번거로운 상황이 생긴다. 따라서 세션의 경우 클라이언트가 가장 최근 서버에 요청한 시간을 기준으로 세션의 유효기간을 최신화하는 방식(타임아웃)을 사용하여 문제를 해결한다.

```java
applicaiton.properties 설정

session.setMaxInactiveInterval(1800); //1800초
```