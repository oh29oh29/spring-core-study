## IoC 컨테이너 7부: MessageSource

* 국제화 (18n) 기능을 제공하는 인터페이스
* ApplicationContext extends MessageSource

<br>

messages.properties
```properties
greeting=안녕, {0}
```

messages_en_US.properties
```properties
greeting=Hello, {0}
```

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(messageSource.getMessage("greeting", new String[]{"oh29oh29"}, Locale.getDefault()));
        System.out.println(messageSource.getMessage("greeting", new String[]{"oh29oh29"}, Locale.US));
    }
}
```

출력 결과
```text
안녕, oh29oh29
Hello, oh29oh29
```

<br>

#### 메세지 빈 등록

* 원래는 Message를 빈으로 등록해야 하는데 스프링부트에는 기본적으로 ResourceBundleMessageSource에 의해 리소스 번들을 읽을 수 있도록 되어 있어서 별다른 설정이 필요하지 않다.

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(messageSource.getClass());
    }
}
```

출력 결과
```text
class org.springframework.context.support.ResourceBundleMessageSource
```

<br>

* 수동으로 빈을 등록하려면 아래와 같이 한다.

```java
@SpringBootApplication
public class SpringcoreexampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcoreexampleApplication.class, args);
    }

    @Bean
    public MessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:/messages");
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource;
    }
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(messageSource.getClass());
        System.out.println(messageSource.getMessage("greeting", new String[]{"oh29oh29"}, Locale.getDefault()));
        System.out.println(messageSource.getMessage("greeting", new String[]{"oh29oh29"}, Locale.US));
    }
}
```

출력 결과
```text
class org.springframework.context.support.ReloadableResourceBundleMessageSource
안녕, oh29oh29
Hello, oh29oh29
```
