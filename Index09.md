## IoC 컨테이너 8부: ApplicationEventPublisher

* 이벤트 프로그래밍에 필요한 인터페이스 제공
* 옵저버 패턴 구현체
* ApplicationContext extends ApplicationEventPublisher

<br>

int형 data를 실어서 보낼 수 있는 이벤트 MyEvent를 아래와 같이 정의한 후 이벤트를 ApplicationContext가 발생시킬 수 있다.
```java
public class MyEvent {

    // 이벤트를 발생시킨 소스
    private Object source;
    
    // 이벤트에 담을 데이터
    private int data;

    public MyEvent(Object source, int data) {
        this.source = source;
        this.data = data;
    }

    public Object getSource() {
        return source;
    }

    public int getData() {
        return data;
    }
}
```
```java
@Component
public class MyEventHandler {

    @EventListener
    public void handle(MyEvent myEvent) {
        System.out.println("MyEventHendler. 데이터는 " + myEvent.getData());
    }
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext applicationContext;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        applicationContext.publishEvent(new MyEvent(this, 100));
    }
}
```

출력 결과
```text
MyEventHendler. 데이터는 100
```

<br>

* 동일한 이벤트에 대하여 이벤트 핸들러가 여러가지 일 경우 모두 실행된다.
```java
@Component
public class MyEventHandler {

    @EventListener
    public void handle(MyEvent myEvent) {
        System.out.println("MyEventHendler. 데이터는 " + myEvent.getData());
    }
}
```
```java
@Component
public class AnotherEventHandler {

    @EventListener
    public void handle(MyEvent myEvent) {
        System.out.println("AnotherEventHandler. 데이터는 " + myEvent.getData());
    }
}
```

출력 결과
```text
AnotherEventHandler. 데이터는 100
MyEventHendler. 데이터는 100
```

이때, MyEventHandler와 AnotherEventHandler 실행 순서는 순차적이지만 순서가 정해져 있지는 않다.
순서를 지정하려면 아래와 같이 @Order를 사용한다.

```java
@Component
public class MyEventHandler {

    @EventListener
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public void handle(MyEvent myEvent) {
        System.out.println("MyEventHendler. 데이터는 " + myEvent.getData());
    }
}
```
```java
@Component
public class AnotherEventHandler {

    @EventListener
    @Order(Ordered.HIGHEST_PRECEDENCE + 1)
    public void handle(MyEvent myEvent) {
        System.out.println("AnotherEventHandler. 데이터는 " + myEvent.getData());
    }
}
```
출력 결과
```text
MyEventHendler. 데이터는 100
AnotherEventHandler. 데이터는 100
```

* 비동기 실행을 하고 싶다면 @Async, @EnableAsync와 같이 사용한다.
```java
@Component
public class MyEventHandler {

    @EventListener
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("MyEventHandler. 데이터는 " + myEvent.getData());
    }
}
```
```java
@Component
public class AnotherEventHandler {

    @EventListener
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("AnotherEventHandler. 데이터는 " + myEvent.getData());
    }
}
```
```java
@SpringBootApplication
@EnableAsync
public class SpringcoreexampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcoreexampleApplication.class, args);
    }
    
}
```
출력 결과
```text
Thread[task-1,5,main]
AnotherEventHandler. 데이터는 100
Thread[task-2,5,main]
MyEventHandler. 데이터는 100
```

<br>

#### 스프링이 제공하는 기본 이벤트

* ContextRefreshedEvent: ApplicationContext를 초기화 했거나 refresh했을 때 발생.
* ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작 신호를 받는 시점에 발생
* ContextStoppedEvent: ApplicationContext를 stop()하여 라이프사이클 빈들이 정지 신호를 받은 시점에 발생
* ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈이 소멸되는 시점에 발생.
* RequestHandledEvent: HTTP 요청을 처리했을 때 발생.
