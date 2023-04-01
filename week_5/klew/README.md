검증을 하는 방식은 다양하다. 예를 들면 Map에 에러내용을 담아 모델로 반환하던가, BindingResult를 사용해서 반환하던가, Validator 라는 마커 인터페이스를 구현하여 사용하는 방식이 있다. 또한 애노테이션 기반의 검증방식도 존재한다.

## BindingResult?

검증오류를 보관하는 객체<br>
_BindingResult 는 핸들러 매개변수에서 자신이 검증할 객체 바로 다음에 위치시켜야한다._

- ex: method(@ModelAttribute Item item, BindingResult bindingResult, ...){ ... }

        @PostMapping("/add")
        public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

            //검증 로직
            if (!StringUtils.hasText(item.getItemName())) {
                bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
            }
            if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
                bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000까지 혀용합니다."));
            }
            if (item.getQuantity() == null || item.getQuantity() >= 9999) {
                bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999까지 가능합니다."));
            }
            //복합 룰 검증
            if (item.getPrice() != null && item.getQuantity() != null) {
                int resultPrice = item.getPrice() * item.getQuantity();
                if (resultPrice < 10000) {
                    bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값: " + resultPrice));
                }
            }

            //검증 실패시 다시 입력 폼으로 이동해야 한다.
            if (bindingResult.hasErrors()) {
                log.info("errors = {}", bindingResult);
                return "validation/v2/addForm";
            }

            //검증 성공 로직
            Item savedItem = itemRepository.save(item);
            redirectAttributes.addAttribute("itemId", savedItem.getId());
            redirectAttributes.addAttribute("status", true);
            return "redirect:/validation/v2/items/{itemId}";
        }

### FieldError

BindingResult에 보관되는 오류 객체
2가지 생성자를 제공

        public FieldError(String objectName, String field, String defaultMessage) {}
        필드에 오류가 있으면 FieldError 객체를 생성해서 bindingResult 에 담아두면 된다.

        objectName : @ModelAttribute 이름
        field : 오류가 발생한 필드 이름
        defaultMessage : 오류 기본 메시지

        public FieldError(String objectName, String field, String defaultMessage);

        objectName : 오류가 발생한 객체 이름
        field : 오류 필드
        rejectedValue : 사용자가 입력한 값(거절된 값)
        bindingFailure : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
        codes : 메시지 코드
        arguments : 메시지에서 사용하는 인자
        defaultMessage : 기본 오류 메시지

### ObjectError

public ObjectError(String objectName, String defaultMessage) {}
특정 필드를 넘어서는 오류가 있으면 ObjectError 객체를 생성해서 bindingResult 에 담아두면 된다.<br>

`objectName` : @ModelAttribute 의 이름 <br>
`defaultMessage` : 오류 기본 메시지 <br>

### BindingResult의 rejectValue(), reject() 메서드

        //before
        bindingResult.addError(new FieldError("item", "itemName",item.getItemName(), false, new String[]{"required.item.itemName"}, null, null))
        bindingResult.addError(new FieldError("item", "price", item.getPrice(),false, new String[]{"range.item.price"}, new Object[]{1000, 1000000}, null))

        //after
        bindingResult.rejectValue("itemName", "required");
        bindingResult.rejectValue("price", "range", new Object[]{1000, 1_000_000}, null);

_errors.properties 는 어디서 가져오는 거지?_

        void rejectValue(@Nullable String field,         //오류 필드명
                                        String errorCode,                //MessageResolver를 위한 오류 코드
                                        @Nullable Object[] errorArgs,    //오류 메세지에서 {0}을 치환하기 위한 값
                                        @Nullable String defaultMessage);//오류 메세지를 못찾을 경우 기본 메세지

rejectVaule 의 메서드를 보면 `field` 와 `errorCode` 를 통해서 메세지를 찾아내는 데 스프링에서는 MessageCodesResolver를 통해서 찾아낸다.

# MessageCodesResolver

MessageCodesResolver 인터페이스이고 DefaultMessageCodesResolver 는 기본 구현체이다.

MessageCodesResolver를 reject(), rejectValue() 메서드에서 사용하기 때문에 우리는 편하게 field와 errorCode 만 인수로 넘겨줌으로써 에러내용을 담을 수 있는 것이다.

**FieldError는 rejectValue("itemName", "required")**

        new String[]{"required.item.itemName", "required.itemName", "required.java.lang.String", "required"}

**ObjectError는 reject("totalPriceMin")**

        new String[]{"totalPriceMin.item", "totalPriceMin"}

_DefaultMessageCodesResolver의 기본 메시지 생성 규칙_

- 객체 오류

        1. code + "." + object name
        2. code

- 필드 오류

        필드 오류의 경우 다음 순서로 4가지 메시지 코드 생성

        1. code + "." + object name + "." + field
        2. code + "." + field
        3. code + "." + field type
        4. code

        예 => 오류 코드: typeMismatch, object name "user", field "age", field type: int

        1. "typeMismatch.user.age"
        2. "typeMismatch.age"
        3. "typeMismatch.int"
        4. "typeMismatch"

# Validator 분리

Validator 인터페이스를 구현해서 검증로직을 만들면 추가적으로 애너테이션을 사용하여 검증을 수행할수도 있다.

        @Slf4j
        @Controller
        @RequestMapping("/validation/v2/items")
        @RequiredArgsConstructor
        public class ValidationItemControllerV2 {
                private final ItemValidator itemValidator;

                @InitBinder
                public void init(WebDataBinder dataBinder){
                    dataBinder.addValidators(itemValidator);
                }
        }

addValidators()메서드를 사용해 검증기를 추가하면 해당 컨트롤러에서 검증기 자동 적용이 가능하다.<br>
WebDataBinder에 ItemValidator 검증기를 추가했다면 다음과같이 애너테이션으로 편하게 검증로직을 수행하고 에러내용을 BindingResult에 담을 수 있다.

        @PostMapping("/add")
        public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //검증 실패시 다시 입력 폼으로 이동해야 한다.
        if (bindingResult.hasErrors()) {
            log.info("errors = {}", bindingResult);
            return "validation/v2/addForm";
        }
            ...
        }

- @Validated 애너테이션을 사용해서 Item의 검증로직을 수행해준다.
- WebDataBinder가 @Validated 애너테이션 붙은 요소를 검증하는데 이때 WebDataBinder가 가진 여러 검증기 중에서 어떤 검증기가 실행되야 할 지 찾기위해 구분이 필요한데 이 때 supports() 메서드가 사용된다.

# Bean Validation

어노테이션 형태로 제약 조건을 달아줘서 쉽게 검증할 수 있도록 돕는 API <br>
Bean Validation을 구현한 기술중 일반적으로 사용하는 구현체는 Hibernate Validator이다.<br>

Bean Validation을 사용하기 위해 아래 의존성을 추가해야 한다.

        // Gradle
        implementation 'org.springframework.boot:spring-boot-starter-validation'

        // Maven
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

| 어노테이션 이름                 | 기능                                                                                                       |
| :------------------------------ | :--------------------------------------------------------------------------------------------------------- |
| @AssertFalse                    | 값이 false인지 확인                                                                                        |
| @AssertTrue                     | 값이 true인지 확인                                                                                         |
| @DecimalMax(value=, inclusive=) | value로 설정한 값보다 작은지 확인 (inclusive가 true이면 value 값도 인정)                                   |
| @DecimalMin(value=, inclusive=) | value로 설정한 값보다 큰지 확인 (inclusive가 true이면 value 값도 인정)                                     |
| @Digits(integer=, fraction=)    | 정해진 자릿수 이하인지 확인 (integer는 허용 가능한 정수 자릿수, fraction은 허용 가능한 소수점 이하 자릿수) |
| @Email                          | 이메일 형식인지 확인                                                                                       |
| @Future                         | 해당 시간이 미래인지 확인                                                                                  |
| @FutureOrPresent                | 해당 시간이 현재 또는 미래인지 확인                                                                        |
| @Max(value=)                    | 설정한 값보다 작은지 확인                                                                                  |
| @Min(value=)                    | 설정한 값보다 큰지 확인                                                                                    |
| @NotBlank                       | null이 아니고 한 개 이상의 문자를 포함하는지 확인(공백 제외)                                               |
| @NotEmpty                       | null이 아니고 한 개 이상의 문자를 포함하는지 확인(공백 포함)                                               |
| @NotNull                        | null이 아닌지 확인                                                                                         |
| @Negative                       | 음수인지 확인                                                                                              |
| @NegativeOrZero                 | 음수이거나 0인지 확인                                                                                      |
| @Null                           | null인지 확인                                                                                              |
| @Past                           | 해당 시간이 과거인지 확인                                                                                  |
| @PastOrPresent                  | 해당 시간이 현재 또는 과거인지 확인                                                                        |
| @Pattern(regex=, flags=)        | 해당 정규식을 만족하는지 확인                                                                              |
| @Positive                       | 양수인지 확인                                                                                              |
| @PositiveOrZero                 | 양수이거나 0인지 확인                                                                                      |
| @Size(min=, max=)               | 해당 범위의 값인지 확인(max, min 값 포함)                                                                  |

<br>

**@Valid 와 @Validated의 차이**

검증 항목을 그룹으로 나눠서 검증할 수 있는지, Group Validation이 가능한 지 여부이다. @Valid 에는 groups를 적용할 수 있는 기능이 없다. groups를 사용하려면 @Validated 를
사용해야 한다.
