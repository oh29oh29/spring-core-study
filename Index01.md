## IoC 컨테이너 1부: 스프링 IoC 컨테이너와 빈

#### IoC (Inversion of Control)

* 의존 관계 주입 (Dependency Injection) 이라고도 한다.
* 의존 객체를 직접 만들어 사용하는 것이 아니라, 주입 받아 사용하는 방법이다.

#### IoC 컨테이너

* Bean 중앙 저장소.
* Bean 설정 소스로 부터 Bean 정의를 읽어들이고, Bean을 구성하고 제공한다.
* IoC 컨테이너의 핵심은 [BeanFactory](https://docs.spring.io/spring/docs/5.0.4.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 인터페이스이다. IoC 컨테이너(ApplicationContext의 최상위 인터페이스)
 
#### 빈

* 스프링 IoC 컨테이너가 관리하는 객체
* 장점
    * 의존성 관리
    * 스코프
        * 싱글톤 (default)
        * 프로토타입 / 매번 다른 객
    * 라이프사이클 인터페이스
 
<br>


```java
@Repository
public class BookRepository {

    public Book save(Book book) {
        return null;
    }
}
```
```java
@Service
public class BookService {

    private BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("PostConstruct");
    }
}
```
* BookService 객체가 BookRepository 객체(의존 객체)를 직접 만들어서 사용하는 것이 아니라 생성자에서 주입 받아서 사용하는 것을 IoC (Inversion of Control) 라고 한다.
* @PostConstruct
    * 라이프사이클 콜백
    * 빈이 생성될 때 별도의 초기화 작업을 위해 실행하는 메소드

```java
@Service
public class BookService implements InitializingBean {

    private BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("PostConstruct");
    }
}
```
* @PostConstruct 외에 InitializingBean 인터페이스를 이용해서 afterPropertiesSet을 구현하면 동일하게 빈이 생성될 때 별도의 초기화 작업을 수행한다.
