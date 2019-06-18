## 데이터 바인딩 추상화: PropertyEditor

* [org.springframework.validation.DataBinder](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/DataBinder.html)
* 기술적인 관점: 프로퍼티 값을 타겟 객체에 설정하는 기능
* 사용자 관점: 사용자 입력값을 애플리케이션 도메인 모델에 동적으로 변환해 넣어주는 기능.
* 입력값은 대부분 문자열인데, 그 값을 객체가 가지고 있는 int, long, Boolean, Date 또는 Event, Book과 같은 도메인 타입으로도 변환해서 넣어주는 기능.

<br>

#### [PropertyEditor](https://docs.oracle.com/javase/7/docs/api/java/beans/PropertyEditor.html)

* 스프링 3.0 이전까지 DataBinder가 변환 작업을 위해 사용하던 인터페이스
* thread-safe 하지 않기 때문에 빈으로 등록하지 않아야 한다. (쌍태 정보를 저장하고 있다)
* Object와 String 간의 변환만 할 수 있어, 사용 범위가 제한적이다.

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
public class EventEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
        Event event = (Event) getValue();
        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        setValue(new Event(Integer.parseInt(text)));
    }
}
```
```java
@RestController
public class EventController {

    @InitBinder
    public void init(WebDataBinder webDataBinder) {
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        System.out.println(event);
        return event.getId().toString();
    }
}
```
* 컨트롤러에서 사용할 바인더들을 등록할 수 있는 @InitBinder로 property editor를 등록해준다.

```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class EventControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void getTest() throws Exception {
        mockMvc.perform(get("/event/1"))
                .andExpect(status().isOk())
                .andExpect(content().string("1"));
    }
}
```
* '/event/1' 이라는 path로 들어오는 요청이 있으면 200 응답에 결과값으로 문자열 1을 리턴하는지 확인하는 테스트 코드

<br>

#### 그러나

* 구현하기 쉽지 않고 thread-safe 하지 않기 때문에 빈으로 등록하기에도 위험하다.
* 그래서 스프링 3부터는 다른 데이터 바인딩과 관련된 인터페이스와 기능들이 추가되었다.
* 따라서 이런 방식은 고전적인 방식으로 현재는 다른 방법이 존재한다.
