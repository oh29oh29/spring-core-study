## 스프링 AOP: 프록시 기반 AOP

#### 스프링 AOP 특징

* 프록시 기반의 AOP 구현체
* 스프링 빈에서만 AOP를 적용할 수 있다.
* 모든 AOP 기능을 제공하는 것이 목적이 아니라, 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔한 문제에 대한 해결책을 제공하는 것이 목적이다.

<br>

#### 프록시 패턴

* 기존 코드 변경없이 접근제어 또는 부가기능 추가

```java
public interface EventService {

    void createEvent();

    void publishEvent();

    void deleteEvent();
}
```

* createEvent()와 publishEvent() 의 동작 시간을 알고 싶을 때, 아래와 같이 동일한 코드를 두 메소드에 넣어줘야 한다.

```java
@Service
public class SimpleEventService implements EventService {

    @Override
    public void createEvent() {

        long begin = System.currentTimeMillis();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Created an event");

        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {

        long begin = System.currentTimeMillis();

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Published an event");

        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
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
        eventService.createEvent();
        eventService.publishEvent();
        eventService.deleteEvent();
    }
}
```

출력 결과
```text
Created an event
1003
Published an event
2004
Delete an event
```

* 이와 같은 방법은 중복 코드가 생기고 메소드 본연의 기능 구현뿐만 아니라 부가 기능까지 한 메소드에 작성해야 하는 단점이 존재한다.
* 보완하는 방법으로 아래와 같은 방법이 있다.

<br>

```java
@Service
public class SimpleEventService implements EventService {

    @Override
    public void createEvent() {

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Created an event");
    }

    @Override
    public void publishEvent() {

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Published an event");
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```
```java
@Primary
@Service
public class ProxySimpleEventService implements EventService {

    @Autowired
    SimpleEventService simpleEventService;

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {
        simpleEventService.deleteEvent();
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
        eventService.createEvent();
        eventService.publishEvent();
        eventService.deleteEvent();
    }
}
```

출력 결과
```text
Created an event
1003
Published an event
2004
Delete an event
```

* 하지만 위와 같은 방법 역시 중복 코드가 생기고 많은 코드 작성 비용이 발생한다.

<br>

#### 문제점

* 매번 프록시 클래스를 작성해야 하는가?
* 여러 클래스 여러 메소드에 적용하려면?
* 복잡한 객체들 관계..

<br>

#### 그래서 등장한 것이 스프링 AOP

* 스프링 IoC 컨테이너가 제공하는 기반 시설과 Dynamic 프록시를 사용하여 여러 복잡한 문제를 해결한다.
* Dynamic 프록시: 동적으로 프록시 객체 생성하는 방법
    * 자바가 제공하는 방법은 인터페이스 기반 프록시 생성.
    * CGlib은 클래스 기반 프록시도 지원.
* 스프링 IoC: 기존 빈을 대체하는 동적 프록시 빈을 만들어 등록시켜준다.
    * 클라이언트 코드 변경 없음.
    * [AbstractAutoProxyCreator](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/aop/framework/autoproxy/AbstractAutoProxyCreator.html) implements [BeanPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html) 
