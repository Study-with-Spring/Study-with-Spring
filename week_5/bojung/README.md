## week 5 - Validation

**Ver. 1**

Map을 활용한 key(error):value(error contents) 방식을 사용하여 오류가 발생하는 조건이 만족하는 경우 Error Map에 값을 저장한다. Map에 값이 저장되어있다면 값을 저장하지 않고 다시 상품 등록 페이지를 호출하여 사용자가 입력한 값이 저장되지 않도록 한다.

→ 타입오류 처리가 불가능하다. (컨트롤러에 진입하기도 전에 예외가 발생하여 400 예외 발생)

→뷰템플릿에서 중복 처리가 많다.

**Ver.2**

**BindingResult** 

필드에 오류가 있으면 FieldError객체를 생성해서 bindingResult에 담아둔다.

```java
public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, ...)
// BindingResult bindingResult 파라미터의 위치는 @ModelAttribute 객체 다음에 와야한다.
// BindingResult 는 Model에 자동으로 포함된다.
	...
if (!StringUtils.hasText(item.getItemName())) { 
	bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
}
```

BindingResult가 있으면 @ModelAttribute에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출된다.

@ModelAttribute의 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 FieldError를 생성해서 BindingResult에 넣어준다.

- 오류가 발생하면 고객이 입력한 내용이 모두 사라진다는 문제가 있다.
    
    ModelAttribute에 바인딩되는 시점에 오류가 발생하면 사용자 입력 값을 유지하기 어렵다. 오류가 발생한 경우 사용자 입력 값을 보관하고 오류 발생시 화면에 다시 출력하면 된다.
    

**FieldError**

오류 발생시 사용자 입력 값을 저장하는 기능을 제공하며, 두 가지 생성자를 제공한다.

```java
public FieldError(String objectName, String field, String defaultMessage);
public FieldError(String objectName, String field, @Nullable ObjectrejectedValue, 
	boolean bindingFailure, @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage)
// objectName : 오류가 발생한 객체 이름
// field : 오류 필드
// rejectedValue : 사용자가 입력한 값(거절된 값)
// bindingFailure : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값 codes : 메시지 코드
// arguments : 메시지에서 사용하는 인자
// defaultMessage : 기본 오류 메시지
```

타입 오류로 바인딩에 실패하면 스프링은 ‘FieldError’를 생성하면서 사용자가 입력한 값을 넣어둔다. 그리고 해당 오류를 ‘BindingResult’에 담아서 컨트롤러를 호출한다.

**ObjectError** → FieldError와 같지만 bindingFailure 파라미터가 필요하지 않음.

**Validation 분리**

컨트롤러에서 모든 검증 기능이 있으면 코드가 복잡하고 명확하지 않다. → Validation 분리

1. 스프링이 제공하는 Validator Interface을 구현
2. 스프링빈으로 등록
3. 컨트롤러의 생성자로 Validator 빈 주입
4. 검증이 필요한 위치에서 검증 메서드 사용

**WebDataBinder**

검증이 필요한 위치에서 검증 메서드를 호출하지 않고 스프링의 도움을 받을 수도 있다.

@InitBinder :

init() 함수에 어노테이션을 추가한다.

@Validated :

검증이 필요한 메서드 파라미터 가장 앞에 어노테이션을 추가하면 자동으로 Validator가 추가된다.
