# 컨버터 (Converter)

자바의 타입형변환은 Integer.valueOf(), Integer.parseInt() 등의 메소드를 사용하여 타입을 변환합니다. 그런데 스프링에서는 `@RequestParam` 어노테이션을 사용하여 타입을 변환할 수 있습니다.

```java
@GetMapping("/hello-v2")
public String helloV2(@RequestParam Integer data) {
    System.out.println("data = " + data);

    return "ok";
}
```

HTTP 쿼리 스트링으로 전달하는 data=10 부분에서 10은 숫자 10이 아니라 문자 10인데, 스프링이 제공하는 `@RequestParam` 을 사용하면 이 문자 10을 Integer 타입의 숫자 10으로 받을 수 있습니다. 이것은 스프링이 중간에서 타입을 변환해주기 때문입니다. 또한 `@ModelAttribute` 나 `@PathVariable` 도 마찬가지입니다.

**만약 새로운 타입을 반환하는 컨버터를 개발하고 싶다면 어떻게 구현할까?**

스프링은 확장 가능한 컨버터 인터페이스를 제공하는데 개발자는 추가적인 타입 변환이 필요하다면 원하는 타입의 컨버터 인터페이스를 구현하면 됩니다.

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {
 T convert(S source);
}
```

컨버팅 하는 기능을 구현하여 컨버터가 필요한 구간마다 의존 주입 후, 메서드를 호출하는 것은 너무 귀찮습니다. <br>

그런데 스프링은 `ConversionService` 인터페이스를 통해 컨버팅 기능을 제공합니다. 이 인터페이스는 컨버팅 가능여부와 컨버팅 기능을 제공합니다. <br>

스프링은 ISP(Interface Segregation Principle)를 따르기 때문에 DefaultConversionService -> GenericConversionSerivce -> ConfigurableConversionService에 들어가보면 ConversionService와 ConversionRegistry로 분리된 2개의 인터페이스에 의존하는 것을 볼 수 있습니다. <br>

`ConversionService`는 `canConvert`, `convert`와 같은 메서드가 있어서 실제 타입 변환을 할 수 있도록 해주는 인터페이스이고,
`ConverterRegistry`는 클라이언트가 만든 Converter를 등록할 수 있는 registry 메서드가 포함되어 있습니다.

> `ConversionService` : 컨버터 사용에 초점

> `ConverterRegistry` : 컨버터 등록에 초점

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToIpPortConverter());
        registry.addConverter(new IpPortToStringConverter());
    }
}
```

# 포맷터 (Formatter)

Formatter는 객체와 문자열 간 변환에 특화되어 있습니다. `org.springframework.format.Formatter` 인터페이스는 `Printer<T>`와 `Parser<T>` 인터페이스를 상속 받습니다. <br>

`Converter`와 다르게 `Formatter`는 두 변환 타입 중 한 가지는 String으로 고정되어 있습니다. Number 객체는 Integer, Double 등 숫자 타입들의 부모 객체입니다. NumberFormat.getInstance로 파라미터로 받아온 String 또는 Number 객체를 NumberFormat으로 변환해준다. 이후 parse, format 메서드를 통해 각각 객체, 문자열로 변환해줍니다. <br>

```java
@Slf4j
public class MyNumberFormatter implements Formatter<Number> {
  @Override
  public Number parse(String text, Locale locale) throws ParseException {
    log.info("text={}, locale={}", text, locale);
    NumberFormat format = NumberFormat.getInstance(locale);
    return format.parse(text);
  }

  @Override
  public String print(Number object, Locale locale) {
    log.info("object={}, locale={}", object, locale);
    return NumberFormat.getInstance(locale).format(object);
  }
}
```

`Formatter` 가 `Converter` 처럼 동작할 수 있도록 어댑터 패턴을 사용하는 `DefaultForamttingConversionService` 구현체가 존재합니다. 스프링 부트는 `DefaultForamttingConversionService` 를 상속 받은 `WebConversionService`를 내부적으로 사용합니다. <br>

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToIpPortConverter());
        registry.addConverter(new IpPortToStringConverter());

        registry.addFormatter(new MyNumberFormatter());
    }
}
```

**주의할 점: HttpMessageConverter 는 ConversionService 가 적용되지 않는다.**
