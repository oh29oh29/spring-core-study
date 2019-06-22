## SpEL (스프링 Expression Language)

#### [스프링 EL](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions)

* 객체 그래프를 조회하고 조작하는 기능을 제공.
* [Unified EL](https://docs.oracle.com/javaee/5/tutorial/doc/bnahq.html)과 비슷하지만, 메소드 호출을 지원하며, 문자열 템플릿 기능도 제공.
* OGNL, MVEL, JBOss EL 등 자바에서 사용할 수 있는 여러 EL이 있지만, SpEL은 모든 스프링 프로젝트 전반에 걸쳐 사용할 EL로 만들었다.
* 스프링 3.0 부터 지원.

<br>

#### SpEL 구성

* ExpressionParser parser = new SpelExpressionParser()
* StandardEvaluationContext context = new StandardEvaluationContext(bean)
* Expression expression = parser.parseExpression("SpEL 표현식")
* String value = expression.getValue(context, String.class)

<br>

#### 문법

* \#{"표현식"}
* ${"프로퍼티"}
* 표현식 안에 프로퍼티를 쓸 수 있지만, 프로퍼티 안에 표현식을 쓸 수 없다.
    * \#{${my.data} + 1}
* [레퍼런스](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions-language-ref)

```properties
#application.properties
my.value=100
```
```java
@Component
public class Sample {

    private int data = 200;

    public int getData() {
        return data;
    }
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Value("#{1 + 1}")
    int value;

    @Value("#{'hello ' + 'world'}")
    String greeting;

    @Value("#{1 eq 1}")
    boolean trueOrFalse;

    @Value("hello")
    String hello;

    @Value("${my.value}")
    int myValue;

    @Value("#{${my.value} eq 100}")
    boolean isMyValue100;

    @Value("#{'spring'}")
    String spring;

    @Value("#{sample.data}")
    int sampleData;
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(value);
        System.out.println(greeting);
        System.out.println(trueOrFalse);
        System.out.println(hello);
        System.out.println(myValue);
        System.out.println(isMyValue100);
        System.out.println(spring);
        System.out.println(sampleData);
        
        ExpressionParser parser = new SpelExpressionParser();
        Expression expression = parser.parseExpression("2 + 100");
        Integer value = expression.getValue(Integer.class);
        System.out.println(value);
    }
}
```

출력 결과
```text
2
hello world
true
hello
100
true
spring
200
102
```

<br>

#### 실제로 쓰이는 곳

* @Value annotaion
* @ConditionalOnExpression annotation
* [Spring Security](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/el-access.html)
    * 메소드 시큐리티, @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter
    * XML 인터셉터 URL 설정
* [Spring Data](https://spring.io/blog/2014/07/15/spel-support-in-spring-data-jpa-query-definitions)
    * @Query annotation
* Thymeleaf
