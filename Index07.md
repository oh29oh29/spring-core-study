## IoC 컨테이너 6부: Environment 1부. 프로퍼티 

* 다양한 방법으로 정의할 수 있는 설정값
* Environment의 역할은 프로퍼티 소스 설정 및 프로퍼티 값 가져오기

<br>

#### 프로퍼티 우선순위

* StandardServletEnvironment의 우선순위
    * ServletConfig 매개변수
    * ServletContext 매개변수
    * JNDI (java:comp/env/)
    * JVM 시스템 프로퍼티 (-Dkey="value")
    * JVM 시스템 환경 변수 (운영 체제 환경 변수)

<br>

#### 프로퍼티 설정

1. JVM 시스템 프로퍼티

    ![vm옵션](/images/index07-vmoption.png)
    
    ```java
    @Component
    public class AppRunner implements ApplicationRunner {
        @Autowired
        ApplicationContext ctx;
        
        @Override
        public void run(ApplicationArguments args) throws Exception {
            Environment environment = ctx.getEnvironment();
            System.out.println(environment.getProperty("app.name"));
        }
    }
    ```
    출력 결과
    
    ```text
    spring5
    ```

2. @PropertySource (Environment를 통해 프로퍼티 추가하는 방법)

    app.properties
    ```properties
    app.name=spring
    ```

    ```java
    @SpringBootApplication
    @PropertySource("classpath:/app.properties")
    public class SpringcoreexampleApplication {
    
       public static void main(String[] args) {
           SpringApplication.run(SpringcoreexampleApplication.class, args);
       }
    }
    ```
    ```java
    @Component
    public class AppRunner implements ApplicationRunner {
       @Autowired
       ApplicationContext ctx;
     
       @Value("${app.name}")
       String appName;
    
       @Override
       public void run(ApplicationArguments args) throws Exception {
           Environment environment = ctx.getEnvironment();
           System.out.println(environment.getProperty("app.name"));
           System.out.println(appName);
       }
    }
    ```
    출력 결과
    
    ```text
    spring
    spring
    ```

<br>

#### 스프링부트의 외부 설정 참고

* 기본 프로퍼티 소스 지원 (application.properties)
* 프로파일까지 고려한 계층형 프로퍼티 우선 순위 제공
