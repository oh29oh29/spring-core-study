## IoC 컨테이너 2부: ApplicationContext와 다양한 빈 설정 방법

#### 스프링 IoC 컨테이너의 역할

* 빈 인스턴스 생성
* 의존 관계 설정
* 빈 제공

<br>

#### ClassPathXmlApplicationContext

* xml을 이용한 빈 설정

```java
public class SpringStudyApplication {
    
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        System.out.println(Arrays.toString(beanDefinitionNames)); // [bookService, bookRepository]

        BookService bookService = (BookService) context.getBean("bookService");
        System.out.println(bookService.bookRepository != null); // true
    }
}
```
```java
public class BookService {
    
    private BookRepository bookRepository;
    
    public void setBookRepository(BookRepository bookRepository) {
       this.bookRepository = bookRepository;
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookService"
            class="pe.oh29oh29.springstudy.ApplicationContextAndBean.BookService">
        <property name="bookRepository" ref="bookRepository"></property>
    </bean>

    <bean id="bookRepository"
            class="pe.oh29oh29.springstudy.ApplicationContextAndBean.BookRepository">
    </bean>
</beans>
```
* XML방식은 빈을 하나하나 작성해야 하는 번거러움이 있다.
* 위 방법을 개선한 방법은 아래와 같다.

```java
@Service
public class BookService {

    private BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
       this.bookRepository = bookRepository;
    }
}
```
```java
@Respository
public class BookRepository {
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- base-package 이하 Annotation을 스캔하여 등록 -->
    <context:component-scan base-package="pe.oh29oh29.springstudy.ApplicationContextAndBean"/>

</beans>
```
* component scan base package 이하 annotaion을 찾아서 빈으로 등록한다.

<br>

#### AnnotationConfigApplicationContext

* java 파일을 이용한 빈 설정

```java
public class SpringStudyApplication {
    
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        System.out.println(Arrays.toString(beanDefinitionNames)); // [bookService, bookRepository]

        BookService bookService = (BookService) context.getBean("bookService");
        System.out.println(bookService.bookRepository != null); // true
    }
}
```
```java
@Configuration
public class ApplicationConfig {
    
    @Bean
    public BookRepository bookRepository() {
        return new BookRepository();
    }

    /*
     * BookRepository 주입 방법 1
     * */
    @Bean
    public BookService bookService() {
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository());
        return bookService;
    }

    /*
    * BookRepository 주입 방법 2
    * */
    @Bean
    public BookService bookService2(BookRepository bookRepository) {
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository);
        return bookService;
    }

}
```

* 또는 아래와 같이 @ComponentScan annotation을 사용해서 빈으로 등록할 수 있다.

```java
@Configuration
@ComponentScan(basePackageClasses = SpringStudyApplication.class)
public class ApplicationConfig {
    
}
```

* 스프링부트에서는 아래와 같이 @SpringBootApplication annotation만 붙이면 ApplicationConfig 조차 필요하지 않다.

```java
@SpringBootApplication
public class SpringStudyApplication {

    public static void main(String[] args) {
    }

}
```
