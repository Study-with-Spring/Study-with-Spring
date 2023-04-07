# 스프링 스터디 5주차

---

### 1. 검증이란?

: 사용자로부터 입력 받는 값에 오류를 검증하여 잘못된 값에 대해 안내를 하고, 정정 하는 과정을 통해 정상적인 서비스 이요이 가능하도록 돕는 절차. 프론트 계층에서도 검증 절차가 가능하나 postman이나 웹의 개발자 도구를 통해서도 충분히 우회가 할 수 있기 때문에 서버 계층에서의 검증 절차가 필수적이다. 

[프론트엔드에서"만" 유효성 검사가 문제인 이유](https://jojoldu.tistory.com/157)

### 2. 검증 방법 - 이론

**1) 검증 직접 처리**

: 입력 값을 java 코드로 직접 처리하는 방식으로 각 필드의 오류에 대해 Map<필드, 메시지> 형태로 저장해 두고 해당 에러 메시지를 출력하는 방식으로 작동한다. 실무에서 쓰이는 방식은 아니나 복합 오류 검증의 경우 마땅한 어노테이션이 없어 실무에서도 해당 방법을 사용하는 것이 권장된다고 한다.

→ 한계 : 타입 오류 처리가 안됨. 자료의 범위나 최소, 최대값 검증을 하기 전에 타입이 맞지 않는 경우 바로 오류가 발생하여 타입 오류를 처리할 수 없다는 단점이 있음. 

**2) BindingResult 활용 by 스프링**

```java
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, 
												RedirectAttributes redirectAttributes)
// BindingResult 위치는 @ModelAttribute Item item 뒤에 와야 한다.
//		== BindingResult는 검증해야 할 대상 바로 뒤에 와야 한다.
```

: BindingResult는 스프링에서 제공하는 검증 오류를 보관하는 객체이다. 필드 오류 발생 시 BindingResult 객체에 FiledError 객체를 생성해서 담아두면 되고, 복합 오류 발생시 ObjectError를 생성해서 담으면 된다. 타입 오류 발생시에도 400에러가 발생하는 것이 아니라 오류 메시지를 BindingResult 객체에 담는다.

**3) FiledError, ObjectError by 스프링**

: 해당 객체를 활용해 오류가 발생한 입력 값을 저장해둘 수 있다. 해당 값을 에러메시지에 포함시키면 사용자 입장에서 잘못 입력한 내용에 대해 이해가 쉽기 때문에 유용하게 활용할 수 있다.

```java
public FieldError(String objectName, String field, @Nullable Object
  rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
  Object[] arguments, @Nullable String defaultMessage)
```

**4) rejectValue(), reject() by 스프링**

: rejectValue를 사용하면 FiledError, ObjectError를 직접 생성하지 않고 에러 내용을 BindingResult에 넘길 수 있다. 파라미터 중 하나인 에러코드는 아래 메시지 리졸버와 함께 사용되는 개념으로 아래 내용 참조.

```java
void rejectValue(@Nullable String field, String errorCode,
        @Nullable Object[] errorArgs, @Nullable String defaultMessage);
```

5**) MessageCodeResolver by 스프링**

: 검증 오류 코드를 자동으로 생성하는 인터페이스이며, 구현체로는 DefaultMessageCodesResolver가 있다. FieldError, ObjectError의 파라미터 중 에러메시지 부분에 properties에 정의된 에러 코드를 넘길 수 있는데 rejectValue 내부의 MessageCodeReslover가 자동으로 여러 계층의 에러 코드를 생성해주기 때문에 에러메시지를 구성할때 불필요한 반복을 막을 수 있다.

```java
if (!StringUtils.hasText(item.getItemName())) {
		bindingResult.rejectValue("itemName", "required", "기본: 상품 이름은 필수입니다.");
}

1. rejectValue() 호출
2. MessageCodesResolver를 사용해서 검증 오류 코드로 메시지 코드들을 생성
3. new FiledError()를 생성하면서 메시지 코드들을 보관
4. 프론트에서 에러 메시지 출력
```

**5) Validator 분리**

: Validator 인터페이스를 상속받아 검증 기능을 가진 클래스로 활용할 경우 Validated 어노테이션으로 검증기를 불러올 수 있다.

```java
@PostMapping("/add")
  public String addItemV6(@Validated @ModelAttribute Item item, BindingResult
  bindingResult, RedirectAttributes redirectAttributes)

> javax.validation.@Valid 를 사용하려면 build.gradle 의존관계 추가가 필요하다.
```

### 3. 검증 방법 - 실무

**1) Bean Validation by 스프링**

: Bean Validation이라는 기술 표준이 존재하고 하이버네이트의 validator 구현체가 많이 쓰인다. (ORM과는 관련 없음) 객체의 각 필드에 애노테이션으로 검증 내용을 명시하여 검증 절차를 간편화 해준다.

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'

//Bean Validation 사용시 해당 의존관계 추가해야
```

- 검증 애노테이션
    
    @NotBlank : 빈 값을 허용하지 않음
    
    @NotNull : null 방지
    
    @Range(min = , max =) : 범위 안의 값
    
    @Max : 최대 값 지정
    
- 검증 순서
    1. @ModelAttribute 각 필드에 타입 변환 시도
        1. 성공시 다음 단계로
        2. 실패 시 typeMismatch 로 필드에러 추가
    2. Validator 적용
        
        : 바인딩에 성공한 필드만 Bean Validation 적용, 여기서 바인딩이란 **사용자가 입력한 데이터를 프로그램 내부의 변수에 연결하는 과정**을 말함.
        
    3. 에러 발생시
        
        : 기존에 살펴 본 방식과 마찬가지로 MessageCodesResolver를 통해 다양한 메시지 코드를 전달함
        

**2) Bean Validation - groups**

: 기존 Bean Validation의 경우 검증 대상의 객체가 다른 기능에서 사용되면서, 다른 검증 과정을 거쳐야할 때 문제가 발생한다. 간단한 예시로 User 정보 객체의 경우에도 회원가입과 정보 수정 등 특정 기능에 따라 전달되는 데이터의 종류와 양도 다르기 때문에 하나의 검증 로직만 저장할 수 있는 기존 Bean Validation은 문제가 있다. 이를 보완하는 것이 한 객체에 여러 검증 로직을 추가할 수 있는 기능이 Bean Validation의 groups이다.

```java
1. groups 생성 : 빈 인터페이스
2. @NotBlank( groups = {SaveCheck.class, UpdateCheck.class})
3. 특정 기능 로직에 @validated(SaveCheck.class) 등의 그룹을 지정
							 (@valid에는 그룹 지정이 기능이 없음)
```

**3) 기능별 객체 분리**

: 유저 정보라 한다면 회원가입을 위한 유저 생성 폼 객체, 수정 폼 객체 등으로 구분하여 사용자의 입력값을 받고 이를 User 객체로 변화하는 과정을 거치는 방식으로 각각 다른 검증 방식을 해결할 수 있다. 실무에서도 각 기능에 따라 워낙 다양한 데이터들이 오가기 때문에 객체를 분리하는 방법이 가장 많이 사용된다.