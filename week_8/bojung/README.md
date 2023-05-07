# week 9 - 스프링 타입 컨버터

### 타입 컨버터

```java
public interface Converter<S, T> {
	T convert(S source);
}
//Converts the source object of type S to target T.
//returns the converted object
```

예시)

```java
public class StringToIntegerConverter implements Converter<String, Integer> {
	@Override
	public Integer converter(String source) {
		Integer integer = Integer.valueOf(source);
		return integer;
	}
}
```

위와 같은 타입 컨버터를 등록하고 관리하면서 편리하게 변환 기능을 제공하는 역할을 하는 컨버전 서비스를 제공한다.

### 컨버전 서비스

스프링은 개별 컨버터를 모아두고 그것들을 묶어서 편리하게 사용할 수 있는 기능을 제공한다.

```java
public class ConversionServiceTest {
      @Test
      void conversionService() {
					//등록
          DefaultConversionService conversionService = new DefaultConversionService();
          conversionService.addConverter(new StringToIntegerConverter());
          conversionService.addConverter(new IntegerToStringConverter());
					//사용
					Integer result = conversionService.convert("10", Integer.class);
					Assertions.assertThat(result).isEqualTo(10);
```

이와 같이 사용자는 컨버터를 등록하기만 하면 컨버전 서비스가 적절한 컨버터를 찾아 적용해준다. 타입이 중복되는 컨버터는 어떻게 구분할까?

컨버전 서비스를 등록하는 부분과 사용하는 부분을 분리하고 의존관계 주입을 사용해야 한다. 

스프링은 내부에서 'ConversionService'를 사용해서 타입을 변환한다.

### 스프링 컨버터

```java
@Configuration
  public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addFormatters(FormatterRegistry registry) {
          registry.addConverter(new StringToIntegerConverter());
          registry.addConverter(new IntegerToStringConverter());
          registry.addConverter(new StringToIpPortConverter());
          registry.addConverter(new IpPortToStringConverter());
		}
}
```

스프링에서 ConversionService를 제공한다. WebMvcConfigurer가 제공하는 `addFormatters()`를 사용해서 컨버터를 추가한다.

스프링은 내부에서 수많은 기본 컨버터들을 제공하며, 컨버터를 추가하면 추가한 컨버터가 기본 컨버터보다 높은 우선순위를 가진다.

### 포맷터

객체를 특정한 포멧에 맞추어 문자로 출력하거나 또는 그 반대의 역할을 하는 것에 특화된 기능. 컨버터의 특별한 버전으로 이해하면 된다.

**웹 애플리케이션에서 객체를 문자로, 문자를 객체로 변환하는 예**

화면에 숫자를 출력해야 하는데, Integer String 출력 시점에 숫자 1000 문자 "1,000" 이렇게 1000 단위에 쉼표를 넣어서 출력하거나, 또는 "1,000" 라는 문자를 1000 이라는 숫자로 변경해야 한다. 날짜 객체를 문자인 "2021-01-01 10:50:11" 와 같이 출력하거나 또는 그 반대의 상황

```java
@Override
      public Number parse(String text, Locale locale) throws ParseException {
          log.info("text={}, locale={}", text, locale);
          NumberFormat format = NumberFormat.getInstance(locale);
          return format.parse(text);
```

'1000'처럼 숫자 중간의 쉼표를 적용하려면 자바가 기본으로 제공하는 NumberFormat객체를 사용한다. Locale 정보를 활용해서 나라별로 다른 숫자 포맷을 만들어준다.

parse()를 사용해서 문자를 숫자로 변환한다.

포맷터를 지원하는 Conversion Service : DefaultFomattingConversionService를 사용하면 컨버터처럼 컨버터 서비스에 등록하여 사용할 수 있다.

### 스프링이 제공하는 기본 포맷터

애노테이션 기반으로 원하는 형식을 지정해서 사용할 수 있는 두 가지 포맷터를 제공한다.

- @NumberFormat: 숫자 관련 형식 지정 포맷터 사용
- @DateTimeFormat: 날짜 관련 형식 지정 포맷터 사용

---

### 파일 업로드

`application/x-www-form-urlencoded` 방식은 HTML 폼 데이터를 서버로 전송하는 가장 기본적인 방법이다. 폼에 입력한 전송할 항목을 key:value 형식으로 HTTP Body에 문자로 username=kim&age=20 와 같이 & 로 구분해서 전송한다. 이는 아래의 문제가 있다.

- 바이너리 데이터 전송의 어려움
- 문자와 바이너리를 동시에 전송

이 문제를 해결하기 위해 HTTP는 `multipart/form-data`라는 전송 방식을 제공한다.

![스크린샷 2023-05-07 오후 9.20.08.png](week%209%20-%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A5%E1%84%90%E1%85%A5%20eac7619da8d74ef982373d8a3fc55f50/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2023-05-07_%25EC%2598%25A4%25ED%259B%2584_9.20.08.png)

`multipart/form-data`는 이렇게 각각의 항목을 구분해서 한번에 전송한다.

```java
public String saveFileV1(HttpServletRequest request) throws
ServletException, IOException {
        log.info("request={}", request);

        String itemName = request.getParameter("itemName");
        log.info("itemName={}", itemName);

				//multipart/form-data 전송 방식에서 각각 나누어진 부분을 받아서 확인할 수 있다.
        Collection<Part> parts = request.getParts();
        log.info("parts={}", parts);

        return "upload-form";
}
```

파일을 업로드를 하려면 실제 파일이 저장되는 경로가 필요하다. `application.properties`에  경로를 입력해둔다.

```java
// application.properties
file.dir=파일 업로드 경로 설정(예): /Users/kimyounghan/study/file/
```

### 스프링과 파일 업로드

스프링은 `MultipartFile`이라는 인터페이스로 멀티파트 파일을 매우 편리하게 지원한다.

```java
public String saveFile(@RequestParam String itemName,
                       @RequestParam MultipartFile file,
											HttpServletRequest request) {

		if (!file.isEmpty()) {
				String fullPath = fileDir + file.getOriginalFilename();
				log.info("파일 저장 fullPath={}", fullPath); file.transferTo(new File(fullPath));
		}

		return "upload-form";
}
```

**MultipartFile 주요 메서드**

file.getOriginalFilename() : 업로드 파일 명 

file.transferTo(...) : 파일 저장

핵심은 데이터베이스에 이미지를 전부 올리지 않는다는 것이다. 데이터베이스에는 서버에 보관된 중복된 부분을 제외한 짧은 경로만 저장한다.
