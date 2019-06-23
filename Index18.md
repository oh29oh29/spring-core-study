## 스프링 AOP: @AOP

* 애노테이션 기반의 스프링 @AOP
* 의존성 추가 (spring-boot-starter-aop)

<br>

#### Aspect 정의

* @Aspect
* 빈으로 등록하기 위해 @Component 추가

<br>

#### Pointcut 정의

* @Pointcut(표현식)
* 주요 표현식
    * execution
    * @annotation
    * bean
* 포인트컷 조합
    * &&, ||, !
    
<br>

#### Advice 정의

* @Before
    * target 실행되기 전에 advice 실행
* @AfterReturning
* @AfterThrowing
* @Around

<br>

```java
public interface EventService {

    void createEvent();

    void publishEvent();

    void deleteEvent();
}
```
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
@Component
@Aspect
public class PerfAspect {

    @Around("execution(* pe.oh29oh29..*.EventService.*(..))")
    public Object logPerf(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long begin = System.currentTimeMillis();
        Object proceed = proceedingJoinPoint.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return proceed;
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
1007
Published an event
2005
Delete an event
0
```

* 이렇게 하면 원치 않는 메소드(deleteEvent())까지도 부가 기능이 실행된다.
* 아래와 같이 애노테이션 기반인 @interface를 만들어서 AOP를 구현할 수 있다.

<br> 

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS) // 기본값 CLASS. 애노테이션 정보를 얼마나 유지할 것인가? 클래스 파일까지 유지하겠다.
public @interface PerfLogging {
}
```
```java
@Service
public class SimpleEventService implements EventService {

    @PerfLogging
    @Override
    public void createEvent() {

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Created an event");
    }

    @PerfLogging
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
@Component
@Aspect
public class PerfAspect {

    @Around("@annotation(PerfLogging)")
    public Object logPerf(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long begin = System.currentTimeMillis();
        Object proceed = proceedingJoinPoint.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return proceed;
    }
}
```

출력 결과
```text
Created an event
1003
Published an event
2000
Delete an event
```

<br>

* bean을 지정하면 해당 빈의 모든 메소드에 적용이 된다.

```java
@Component
@Aspect
public class PerfAspect {

    @Around("bean(simpleEventService)")
    public Object logPerf(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long begin = System.currentTimeMillis();
        Object proceed = proceedingJoinPoint.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return proceed;
    }

    @Before("bean(simpleEventService)")
    public void hello() {
        System.out.println("Hello");
    }
}
```

출력 결과
```text
Hello
Created an event
1007
Hello
Published an event
2004
Hello
Delete an event
0
```

<br>

#### 참고

* [https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts)
