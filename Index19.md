## Null-safety

* 스프링 프레임워크 5에 추가된 Null 관련 애노테이션

<br>

#### 애노테이션

* @NonNull
* @Nullable
* @NonNullApi (패키지 레벨 설정)
* @NonNullFields (패키지 레벨 설정)

<br>
    
#### 목적

* 컴파일 시점에 최대한 NullPointerException 발생을 방지하기 위한 목적


```java
import org.springframework.lang.NonNull;
import org.springframework.stereotype.Service;

@Service
public class EventService {

    @NonNull
    public String createEvent(@NonNull String name) {
        return "hello " + name;
    }
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent(null);
    }
}
```

* 이렇게 해도 아무런 error나 warning이 나타나지 않는다.
* 인텔리J 설정을 바꿔줘야 한다.

<br>

![인텔리J 설정1](/images/index19-intellij-null-config01.png)

![인텔리J 설정2](/images/index19-intellij-null-config02.png)

이미지와 같이 설정을 해주면 eventService.createEvent(null) 의 매개변수 null에 아래와 같은 경고 문구가 표시된다.

```text
Passing 'null' argument to parameter annotated as @NotNull less... (⌘F1) 
Inspection info: This inspection analyzes method control and data flow to report possible conditions that are always true or false, expressions whose value is statically proven to be constant, and situations that can lead to nullability contract violations.
Variables, method parameters and return values marked as @Nullable or @NotNull are treated as nullable (or not-null, respectively) and used during the analysis to check nullability contracts, e.g. report NullPointerException (NPE) errors that might be produced.
More complex contracts can be defined using @Contract annotation, for example:
@Contract("_, null -> null") — method returns null if its second argument is null @Contract("_, null -> null; _, !null -> !null") — method returns null if its second argument is null and not-null otherwise @Contract("true -> fail") — a typical assertFalse method which throws an exception if true is passed to it 
The inspection can be configured to use custom @Nullable
@NotNull annotations (by default the ones from annotations.jar will be used)
```
