@RestController => String을 리턴하면 html페이지에도 그대로 반영

# 요청 매핑

**@RequestMapping**

클라이언트 요청에 정보를 어떤 Controller가 처리할지를 매핑하기 위한 어노테이션입니다.@RequestMapping에 URL을 포함하여 해당 Controller 클래스에 명시하여 사용하는데, 브라우저에서 해당 URL이 호출되면 Controller 내부의 메서드가 호출된다.

**@RestController**

스프링이 자동으로 스프링 빈으로 등록한다. @RestController 어노테이션 내부에 @Component 어노테이션이 포함되어 있어 컴포넌트 스캔의 대싱이 되기 때문이다.

Spring MVC에서 어노테이션 기반 컨트롤러로 인식하게 된다.

<br>

# 메시지

**메시지**

HTML 페이지의 '상품명' 같은 이름을 '상품이름'으로 바꿀 때 이를 하드코딩 해야하는 어려움이 있다.<br>
이런 메시지들을 일일이 고치는 번거러움을 해결하기 위해 한곳에서 고치는 것을 "메시지" 기능이라고 한다.

> messages_ko.properties
>
> > item=상품 <br>
> > item.id=상품 ID <br>
> > item.itemName=상품명 <br>
> > item.price=가격 <br>
> > item.quantity=수량 <br>

<br>

**국제화**

"messages_en.properties" 처럼 다른 언어를 써서 메시지기능을 관리할 수 있다.
