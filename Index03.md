## IoC 컨테이너 3부: @Autowired

* 필요한 의존 객체의 타입에 해당하는 빈을 찾아 주입한다.

<br>

#### @Autowired(required = false)

* 빈으로 등록되어 있지 않은 객체라면 주입받지 않는다.

```java
@Service
public class BookService {

    private BookRepository bookRepository;

    @Autowired(required = false)
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```
```java
public class BookRepository {

}
```

* required의 기본값은 true이며, 정상적으로 모두 빈으로 등록되어 있을 경우에는 아래와 같이 작성하면 된다.
```java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

}
```
```java
@Repository
public class BookRepository {

}
```

<br>

#### 동일한 타입의 빈이 여러 개인 경우

* 어느 빈을 주입해야 하는지 스프링에서 알지 못하기 때문에 오류가 발생한다.
* 기본적으로 주입해야 하는 빈에 @Primary를 붙이거나 @Qualifier("빈 ID")를 사용해서 원하는 빈을 주입받는다.

```java
public interface BookRepository {

}
```
```java
@Repository
public class MyBookRepository implements BookRepository {
    
}
```
```java
@Repository
public class YourBookRepository implements BookRepository {

}
```
위와 같이 동일한 타입의 인터페이스를 구현한 두개의 빈이 존재할 때 아래와 같이 빈을 주입받으려고 하면 'Field bookRepository in pe.oh29oh29.springbasicstudy.autowired.BookService required a single bean, but 2 were found' 라는 오류가 발생한다.
어느 빈을 주입해야 하는지 스프링에서 알지 못해서 발생하는 오류이다.
```java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

}
```

<br>

##### @Primary

* @Primary는 동일한 타입의 여러 빈 중에 해당 빈을 기본으로 주입하도록 한다.

```java
@Repository @Primary
public class MyBookRepository implements BookRepository {
    
}
```
```java
@Repository
public class YourBookRepository implements BookRepository {

}
```
```java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository; // MyBookRepository

}
```

<br>

##### @Qulifier

* @Qualifier("빈 ID") : 해당 ID를 가진 빈을 주입하라는 annotation으로 빈 ID는 빈으로 등록된 클래스명(첫 글자가 lower case)이다.

```java
@Repository
public class MyBookRepository implements BookRepository {

}
```
```java
@Repository
public class YourBookRepository implements BookRepository {

}
```
```java
@Service
public class BookService {

    @Autowired @Qualifier("myBookRepository")
    private BookRepository bookRepository;

}
```

<br>
    
#### [BeanPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)

* 빈 라이프사이클 콜백.
* BeanFactory가 빈으로 등록되어 있는 BeanPostProcessor를 찾아서 해당 로직을 각 빈에 적용한다.
