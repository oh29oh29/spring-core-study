## Validation 추상화

* [org.springframework.validation.Validator](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/Validator.html)
* 애플리케이션에서 사용하는 객체 검증용 인터페이스
* 스프링 내부에서 많이 사용하는 인터페이스.

<br>

#### 특정

* 어떠한 계층과도 관계가 없다. 모든 계층(웹, 서비스, 데이터)에서 사용해도 좋다.
* 구현체 중 하나로 JSR-303(Bean Validation 1.0)과 JSR-349(Bean Validation 1.1)을 지원한다 ([LocalValidatorFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/validation/beanvalidation/LocalValidatorFactoryBean.html))
* DataBinder에 들어가 바인딩 할 때 같이 사용되기도 한다.

<br>

#### 인터페이스

* boolean supports(Class clazz): 어떤 타입의 객체를 검증할 때 사용할 것인지 결정함.
* void validate(Object obj, Errors e): 실제 검증 로직을 이 안에서 구현
    * 구현할 때 ValidationUtils 사용하면 편리함.
    
```java
public class Event {

    Integer id;

    String title;

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
}
```
```java
public class EventValidator implements Validator {
    @Override
    public boolean supports(Class<?> aClass) {
        return Event.class.equals(aClass);
    }

    @Override
    public void validate(Object o, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "title", "notempty", "empty title is not allowed");
    }
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Event event = new Event();
        EventValidator eventValidator = new EventValidator();
        Errors errors = new BeanPropertyBindingResult(event, "event");

        eventValidator.validate(event, errors);

        System.out.println(errors.hasErrors());

        errors.getAllErrors().forEach(e -> {
            System.out.println("=== error code ===");
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });
    }
}
```
* Errors 타입의 구현체인 BeanPropertyBindingResult는 스프링 MVC가 자동으로 생성해서 파라미터로 전달해주기 때문에 실제로 직접 사용할 일은 없다.

출력 결과
```text
true
=== error code ===
notempty.event.title
notempty.title
notempty.java.lang.String
notempty
empty title is not allowed
```

* 코드 상으로는 notempty 에러 코드만 만들어줬지만 Validator에서 notempty.event.title, notempty.title, notempty.java.lang.String와 같은 에러 코드를 추가로 생성해준다.

<br>

#### 스프링 부트 2.0.5 이상 버전을 사용할 때

* 스프링이 제공해주는 LocalValidatorFactoryBean을 빈으로 자동 등록해준다.
* JSR-380(Bean Validation 2.0.1) 구현체로 hibernate-validator 사용.
* [https://beanvalidation.org](https://beanvalidation.org)

```java
import javax.validation.constraints.Email;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotEmpty;

public class Event {

    Integer id;

    @NotEmpty
    String title;

    @Min(0)
    Integer limit;

    @Email
    String email;

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

    public Integer getLimit() {
        return limit;
    }

    public void setLimit(Integer limit) {
        this.limit = limit;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    Validator validator;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(validator.getClass());

        Event event = new Event();
        event.setLimit(-1);
        event.setEmail("test");
        
        Errors errors = new BeanPropertyBindingResult(event, "event");

        validator.validate(event, errors);

        System.out.println(errors.hasErrors());

        errors.getAllErrors().forEach(e -> {
            System.out.println("=== error code ===");
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });
    }
}
```

출력 결과
```text
class org.springframework.validation.beanvalidation.LocalValidatorFactoryBean
true
=== error code ===
Min.event.limit
Min.limit
Min.java.lang.Integer
Min
반드시 0보다 같거나 커야 합니다.
=== error code ===
Email.event.email
Email.email
Email.java.lang.String
Email
이메일 주소가 유효하지 않습니다.
=== error code ===
NotEmpty.event.title
NotEmpty.title
NotEmpty.java.lang.String
NotEmpty
반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다.
```
