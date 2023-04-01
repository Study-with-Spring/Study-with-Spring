# 스프링 스터디 4주차

---

### 1. 로깅

- System.out.println() 대신 로깅 라이브러리를 사용
- SLF4J : Logback, log4J, log6j 를 모두 통합한 것
- SLF4J(인터페이스) → Logback(구현체)
- @Slf4j 롬복으로 사용 가능
    
    ```jsx
    log.info("hello"); // 로깅 사용 예시
    log.debug("data = {}", data);
    ```
    
- 로그 출력 레벨을 지정 (기본 : info)

### 2. 요청 매핑

- @RestController
    
    → 기존 @Controller는 반환 값이 String일 경우 뷰 이름으로 자동 인식되는 반변 RestController는 반환값을 HTTP 메시지 바디에 바로 입력한다.
    
    → method를 지정해주지 않으면 모든 HTTP 메소드에 반응함
    
- 특정 헤더 조건 매핑
    
    → 특정 헤더가 있을때에만 작동
    
      `@GetMapping(value = "/mapping-param", params = "mode=debug")`
    
- @ModelAttribute
    
    : 객체를 생성하고 입력받은 값을 넣어주는 과정 → @ModelAttribute 가 자동화