## IoC 컨테이너 6부: Environment 1부. 프로파일 

* ApplicationContext는 BeanFactory의 역할만 하는것이 아니라 다양한 기능을 갖고 있다.
* 그 중 하나가 Environment의 프로파일 기능이다.

<br>

#### 프로파일

* 빈들의 묶음
* 특정 환경(ex. local, test, production)에 따라 다른 빈들을 등록 또는 사용해야 하는 경우 구분을 하기 위해 프로파일이라는 기능이 존재한다.

```java
@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = ctx.getEnvironment();
        System.out.println(Arrays.toString(environment.getDefaultProfiles()));
    }
}
```

출력 결과

```text
[default]
```

* defaultProfile은 특정 프로파일을 설정과 관계 없이 기본적으로 설정되는 프로파일이다.

<br>

#### 프로파일 정의하기

* 다음은 특정 프로파일에서만 빈을 등록하는 예시

    * @Configuration에서 프로파일 설정 및 빈 등록

    ```java
    @Configuration
    @Profile("test")
    public class TestConfiguration {
    
        @Bean
        public BookRepository bookRepository() {
            return new BookRepository();
        }
    }
    ```

    * @Component에서 프로파일 설정

    ```java
    @Repository
    @Profile("test")
    public class BookRepository {
    
    }
    ```

    * 메소드(@Bean)에서 프로파일 설정

    ```java
    @Configuration
    public class TestConfiguration {
    
        @Bean
        @Profile("test")
        public BookRepository bookRepository() {
            return new BookRepository();
        }
    }
    ```

<br>

#### 프로파일 표현식

* ! : not
* & : and
* | : or
