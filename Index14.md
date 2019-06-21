## 데이터 바인딩 추상화: Converter와 Formatter

#### [Converter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/converter/Converter.html)

* S 타입을 T 타입으로 변환할 수 있는 매우 일반적인 변환기
* 상태 정보 없음 == Stateless == Thread-safe
* [ConverterRegistry](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/converter/ConverterRegistry.html)에 등록해서 사용

```java
public class Event {

    private Integer id;

    private String title;

    public Event(Integer id) {
        this.id = id;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return "Event{" +
                "id=" + id +
                ", title='" + title + '\'' +
                '}';
    }
}
```
```java
import org.springframework.core.convert.converter.Converter;
public class EventConverter {

    @Component
    public static class StringToEventConverter implements Converter<String, Event> {

        @Override
        public Event convert(String s) {
            return new Event(Integer.parseInt(s));
        }
    }
}
```
스프링 웹 MVC 환경에서는 아래와 같은 Configuration을 작성한다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new EventConverter.StringToEventConverter());
    }
}
```
```java
@RestController
public class EventController {

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        return event.getId().toString();
    }
}
```

<br>

#### [Formatter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/format/Formatter.html)

* PropertyEditor 대체제
* Object와 String 간의 변환을 담당한다.
* 문자열을 Locale에 따라 다국화하는 기능도 제공한다. (Optional)
* [FormatterRegistry](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/format/FormatterRegistry.html)에 등록해서 사용

```java
import org.springframework.format.Formatter;
public class EventFormatter implements Formatter<Event> {

    @Override
    public Event parse(String s, Locale locale) throws ParseException {
        return new Event(Integer.parseInt(s));
    }

    @Override
    public String print(Event event, Locale locale) {
        return event.getId().toString();
    }
}
```
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new EventFormatter());
    }
}
```

<br>

#### [ConversionService](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/ConversionService.html)

* 실제 변환 작업은 이 인터페이스를 통해서 Thread-safe하게 사용할 수 있다.
* 스프링 MVC, 빈(value) 설정, SpEL에서 사용한다.
* class DefaultFormattingConversionService
    * implements FormatterRegistry, ConversionService
    * 여러 기본 Converter와 Fomatter를 등록해준다.
    
<br>
    
#### 스프링 부트에서는

* 웹 어플리케이션인 경우에 DefaultFormattingConversionService를 상속하여 만든 WebConversionService를 빈으로 등록해준다.
* 스프링 웹 MVC 환경에서 작성했었던 Configuration이 필요없고 Formatter와 Converter 빈을 찾아 자동으로 등록해준다.
* 기본적으로 등록된 Converter와 Fomatter는 아래와 같다.

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ConversionService conversionService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(conversionService);
    }
}
```
출력 결과
```text
ConversionService converters =
	@org.springframework.format.annotation.DateTimeFormat java.lang.Long -> java.lang.String: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@3a45c42a,@org.springframework.format.annotation.NumberFormat java.lang.Long -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	@org.springframework.format.annotation.DateTimeFormat java.time.LocalDate -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.time.LocalDate -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@28fa700e
	@org.springframework.format.annotation.DateTimeFormat java.time.LocalDateTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.time.LocalDateTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@11963225
	@org.springframework.format.annotation.DateTimeFormat java.time.LocalTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.time.LocalTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@e041f0c
	@org.springframework.format.annotation.DateTimeFormat java.time.OffsetDateTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.time.OffsetDateTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@61a5b4ae
	@org.springframework.format.annotation.DateTimeFormat java.time.OffsetTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.time.OffsetTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@5b69fd74
	@org.springframework.format.annotation.DateTimeFormat java.time.ZonedDateTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.time.ZonedDateTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@11ee02f8
	@org.springframework.format.annotation.DateTimeFormat java.util.Calendar -> java.lang.String: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@3a45c42a
	@org.springframework.format.annotation.DateTimeFormat java.util.Date -> java.lang.String: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@3a45c42a
	@org.springframework.format.annotation.NumberFormat java.lang.Byte -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	@org.springframework.format.annotation.NumberFormat java.lang.Double -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	@org.springframework.format.annotation.NumberFormat java.lang.Float -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	@org.springframework.format.annotation.NumberFormat java.lang.Integer -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	@org.springframework.format.annotation.NumberFormat java.lang.Short -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	@org.springframework.format.annotation.NumberFormat java.math.BigDecimal -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	@org.springframework.format.annotation.NumberFormat java.math.BigInteger -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.Boolean -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@7bd69e82
	java.lang.Character -> java.lang.Number : org.springframework.core.convert.support.CharacterToNumberFactory@4ced35ed
	java.lang.Character -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@1e392345
	java.lang.Enum -> java.lang.Integer : org.springframework.core.convert.support.EnumToIntegerConverter@27dc79f7
	java.lang.Enum -> java.lang.String : org.springframework.core.convert.support.EnumToStringConverter@51b01960
	java.lang.Integer -> java.lang.Enum : org.springframework.core.convert.support.IntegerToEnumConverterFactory@6831d8fd
	java.lang.Long -> java.time.Instant : org.springframework.format.datetime.standard.DateTimeConverters$LongToInstantConverter@622ef26a
	java.lang.Long -> java.util.Calendar : org.springframework.format.datetime.DateFormatterRegistrar$LongToCalendarConverter@10ad20cb,java.lang.Long -> java.util.Calendar : org.springframework.format.datetime.DateFormatterRegistrar$LongToCalendarConverter@7dd712e8
	java.lang.Long -> java.util.Date : org.springframework.format.datetime.DateFormatterRegistrar$LongToDateConverter@70f43b45,java.lang.Long -> java.util.Date : org.springframework.format.datetime.DateFormatterRegistrar$LongToDateConverter@26d10f2e
	java.lang.Number -> java.lang.Character : org.springframework.core.convert.support.NumberToCharacterConverter@12f3afb5
	java.lang.Number -> java.lang.Number : org.springframework.core.convert.support.NumberToNumberConverterFactory@4d4d8fcf
	java.lang.Number -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@6f0628de
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.lang.Long: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@3a45c42a,java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Long: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.LocalDate: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.lang.String -> java.time.LocalDate: org.springframework.format.datetime.standard.TemporalAccessorParser@3d526ad9
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.LocalDateTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.lang.String -> java.time.LocalDateTime: org.springframework.format.datetime.standard.TemporalAccessorParser@3f3c966c
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.LocalTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.lang.String -> java.time.LocalTime: org.springframework.format.datetime.standard.TemporalAccessorParser@6a175569
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.OffsetDateTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.lang.String -> java.time.OffsetDateTime: org.springframework.format.datetime.standard.TemporalAccessorParser@3a71c100
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.OffsetTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.lang.String -> java.time.OffsetTime: org.springframework.format.datetime.standard.TemporalAccessorParser@f325091
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.ZonedDateTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@5f577419,java.lang.String -> java.time.ZonedDateTime: org.springframework.format.datetime.standard.TemporalAccessorParser@4102b1b1
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.util.Calendar: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@3a45c42a
	java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.util.Date: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@3a45c42a
	java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Byte: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Double: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Float: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Integer: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Short: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> @org.springframework.format.annotation.NumberFormat java.math.BigDecimal: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> @org.springframework.format.annotation.NumberFormat java.math.BigInteger: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@5af5def9
	java.lang.String -> java.lang.Boolean : org.springframework.core.convert.support.StringToBooleanConverter@2c22a348
	java.lang.String -> java.lang.Character : org.springframework.core.convert.support.StringToCharacterConverter@3fabf088
	java.lang.String -> java.lang.Enum : org.springframework.core.convert.support.StringToEnumConverterFactory@74d7184a
	java.lang.String -> java.lang.Number : org.springframework.core.convert.support.StringToNumberConverterFactory@610db97e
	java.lang.String -> java.nio.charset.Charset : org.springframework.core.convert.support.StringToCharsetConverter@5cbf9e9f
	java.lang.String -> java.time.Duration: org.springframework.format.datetime.standard.DurationFormatter@63a5e46c
	java.lang.String -> java.time.Instant: org.springframework.format.datetime.standard.InstantFormatter@437e951d
	java.lang.String -> java.time.Month: org.springframework.format.datetime.standard.MonthFormatter@49ef32e0
	java.lang.String -> java.time.MonthDay: org.springframework.format.datetime.standard.MonthDayFormatter@6bd51ed8
	java.lang.String -> java.time.Period: org.springframework.format.datetime.standard.PeriodFormatter@77b325b3
	java.lang.String -> java.time.Year: org.springframework.format.datetime.standard.YearFormatter@7e8e8651
	java.lang.String -> java.time.YearMonth: org.springframework.format.datetime.standard.YearMonthFormatter@271f18d3
	java.lang.String -> java.util.Currency : org.springframework.core.convert.support.StringToCurrencyConverter@5a2f016d
	java.lang.String -> java.util.Locale : org.springframework.core.convert.support.StringToLocaleConverter@6b85300e
	java.lang.String -> java.util.Properties : org.springframework.core.convert.support.StringToPropertiesConverter@3ad394e6
	java.lang.String -> java.util.TimeZone : org.springframework.core.convert.support.StringToTimeZoneConverter@4acf72b6
	java.lang.String -> java.util.UUID : org.springframework.core.convert.support.StringToUUIDConverter@42deb43a
	java.nio.charset.Charset -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@18e8473e
	java.time.Duration -> java.lang.String : org.springframework.format.datetime.standard.DurationFormatter@63a5e46c
	java.time.Instant -> java.lang.Long : org.springframework.format.datetime.standard.DateTimeConverters$InstantToLongConverter@41de5768
	java.time.Instant -> java.lang.String : org.springframework.format.datetime.standard.InstantFormatter@437e951d
	java.time.LocalDateTime -> java.time.LocalDate : org.springframework.format.datetime.standard.DateTimeConverters$LocalDateTimeToLocalDateConverter@2c282004
	java.time.LocalDateTime -> java.time.LocalTime : org.springframework.format.datetime.standard.DateTimeConverters$LocalDateTimeToLocalTimeConverter@22ee2d0
	java.time.Month -> java.lang.String : org.springframework.format.datetime.standard.MonthFormatter@49ef32e0
	java.time.MonthDay -> java.lang.String : org.springframework.format.datetime.standard.MonthDayFormatter@6bd51ed8
	java.time.OffsetDateTime -> java.time.Instant : org.springframework.format.datetime.standard.DateTimeConverters$OffsetDateTimeToInstantConverter@6a1ebcff
	java.time.OffsetDateTime -> java.time.LocalDate : org.springframework.format.datetime.standard.DateTimeConverters$OffsetDateTimeToLocalDateConverter@4b770e40
	java.time.OffsetDateTime -> java.time.LocalDateTime : org.springframework.format.datetime.standard.DateTimeConverters$OffsetDateTimeToLocalDateTimeConverter@54a3ab8f
	java.time.OffsetDateTime -> java.time.LocalTime : org.springframework.format.datetime.standard.DateTimeConverters$OffsetDateTimeToLocalTimeConverter@78e16155
	java.time.OffsetDateTime -> java.time.ZonedDateTime : org.springframework.format.datetime.standard.DateTimeConverters$OffsetDateTimeToZonedDateTimeConverter@1968a49c
	java.time.Period -> java.lang.String : org.springframework.format.datetime.standard.PeriodFormatter@77b325b3
	java.time.Year -> java.lang.String : org.springframework.format.datetime.standard.YearFormatter@7e8e8651
	java.time.YearMonth -> java.lang.String : org.springframework.format.datetime.standard.YearMonthFormatter@271f18d3
	java.time.ZoneId -> java.util.TimeZone : org.springframework.core.convert.support.ZoneIdToTimeZoneConverter@7561db12
	java.time.ZonedDateTime -> java.time.Instant : org.springframework.format.datetime.standard.DateTimeConverters$ZonedDateTimeToInstantConverter@3c1e3314
	java.time.ZonedDateTime -> java.time.LocalDate : org.springframework.format.datetime.standard.DateTimeConverters$ZonedDateTimeToLocalDateConverter@7bfc3126
	java.time.ZonedDateTime -> java.time.LocalDateTime : org.springframework.format.datetime.standard.DateTimeConverters$ZonedDateTimeToLocalDateTimeConverter@53bc1328
	java.time.ZonedDateTime -> java.time.LocalTime : org.springframework.format.datetime.standard.DateTimeConverters$ZonedDateTimeToLocalTimeConverter@3e792ce3
	java.time.ZonedDateTime -> java.time.OffsetDateTime : org.springframework.format.datetime.standard.DateTimeConverters$ZonedDateTimeToOffsetDateTimeConverter@26f143ed
	java.time.ZonedDateTime -> java.util.Calendar : org.springframework.core.convert.support.ZonedDateTimeToCalendarConverter@3301500b
	java.util.Calendar -> java.lang.Long : org.springframework.format.datetime.DateFormatterRegistrar$CalendarToLongConverter@32c0915e,java.util.Calendar -> java.lang.Long : org.springframework.format.datetime.DateFormatterRegistrar$CalendarToLongConverter@106faf11
	java.util.Calendar -> java.time.Instant : org.springframework.format.datetime.standard.DateTimeConverters$CalendarToInstantConverter@6b739528
	java.util.Calendar -> java.time.LocalDate : org.springframework.format.datetime.standard.DateTimeConverters$CalendarToLocalDateConverter@c20be82
	java.util.Calendar -> java.time.LocalDateTime : org.springframework.format.datetime.standard.DateTimeConverters$CalendarToLocalDateTimeConverter@3ef41c66
	java.util.Calendar -> java.time.LocalTime : org.springframework.format.datetime.standard.DateTimeConverters$CalendarToLocalTimeConverter@13c612bd
	java.util.Calendar -> java.time.OffsetDateTime : org.springframework.format.datetime.standard.DateTimeConverters$CalendarToOffsetDateTimeConverter@50b0bc4c
	java.util.Calendar -> java.time.ZonedDateTime : org.springframework.format.datetime.standard.DateTimeConverters$CalendarToZonedDateTimeConverter@19868320
	java.util.Calendar -> java.util.Date : org.springframework.format.datetime.DateFormatterRegistrar$CalendarToDateConverter@7692cd34,java.util.Calendar -> java.util.Date : org.springframework.format.datetime.DateFormatterRegistrar$CalendarToDateConverter@33aa93c
	java.util.Currency -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@1a38ba58
	java.util.Date -> java.lang.Long : org.springframework.format.datetime.DateFormatterRegistrar$DateToLongConverter@36dce7ed,java.util.Date -> java.lang.Long : org.springframework.format.datetime.DateFormatterRegistrar$DateToLongConverter@47a64f7d
	java.util.Date -> java.util.Calendar : org.springframework.format.datetime.DateFormatterRegistrar$DateToCalendarConverter@33d05366,java.util.Date -> java.util.Calendar : org.springframework.format.datetime.DateFormatterRegistrar$DateToCalendarConverter@27a0a5a2
	java.util.Locale -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@3aaf4f07
	java.util.Properties -> java.lang.String : org.springframework.core.convert.support.PropertiesToStringConverter@6058e535
	java.util.UUID -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@1deb2c43
	org.springframework.core.convert.support.ArrayToArrayConverter@2b27cc70
	org.springframework.core.convert.support.ArrayToCollectionConverter@3bb9efbc
	org.springframework.core.convert.support.ArrayToObjectConverter@50d68830
	org.springframework.core.convert.support.ArrayToStringConverter@79f227a9
	org.springframework.core.convert.support.ByteBufferConverter@a23a01d
	org.springframework.core.convert.support.ByteBufferConverter@a23a01d
	org.springframework.core.convert.support.ByteBufferConverter@a23a01d
	org.springframework.core.convert.support.ByteBufferConverter@a23a01d
	org.springframework.core.convert.support.CollectionToArrayConverter@1cefc4b3
	org.springframework.core.convert.support.CollectionToCollectionConverter@6f6a7463
	org.springframework.core.convert.support.CollectionToObjectConverter@6754ef00
	org.springframework.core.convert.support.CollectionToStringConverter@7674a051
	org.springframework.core.convert.support.FallbackObjectToStringConverter@6e9c413e
	org.springframework.core.convert.support.IdToEntityConverter@24b52d3e,org.springframework.core.convert.support.ObjectToObjectConverter@15deb1dc
	org.springframework.core.convert.support.MapToMapConverter@1bdaa23d
	org.springframework.core.convert.support.ObjectToArrayConverter@1e53135d
	org.springframework.core.convert.support.ObjectToCollectionConverter@619bd14c
	org.springframework.core.convert.support.ObjectToOptionalConverter@57a4d5ee
	org.springframework.core.convert.support.ObjectToOptionalConverter@57a4d5ee
	org.springframework.core.convert.support.ObjectToOptionalConverter@57a4d5ee
	org.springframework.core.convert.support.StreamConverter@323e8306
	org.springframework.core.convert.support.StreamConverter@323e8306
	org.springframework.core.convert.support.StreamConverter@323e8306
	org.springframework.core.convert.support.StreamConverter@323e8306
	org.springframework.core.convert.support.StringToArrayConverter@6ca320ab
	org.springframework.core.convert.support.StringToCollectionConverter@3a7704c
```
