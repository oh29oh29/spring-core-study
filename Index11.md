## Resource 추상화

* [org.springframework.core.io.Resource](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/Resource.html)
* java.net.URL을 추상화 한 것.
* 스프링 내부에서 많이 사용하는 인터페이스.

<br>

#### 추상화한 이유

* 클래스패스 기준으로 리소스 읽어오는 기능 부재
* ServletContext 기준으로 상대 경로로 읽어오는 기능 부재
* 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수는 있지만 구현이 복잡하고 편의성 메소드가 부족하다.

<br>

#### 구현체

* UrlResource: [java.net.URL](https://docs.oracle.com/javase/7/docs/api/java/net/URL.html) 참고, 기본으로 지원하는 프로토콜은 http, https, ftp, file, jar 가 있다.
* ClassPathResource: 지원하는 접두어로 classpath: 가 있다.
* FileSystemResource
* ServletContextResource: 웹 애플리케이션 루트에서 상대 경로로 리소스를 찾는다.

<br>

#### 리소스 읽어오기

* Resource의 타입은 location 문자열과 ApplicationContext의 타입에 따라 결정된다.
    * ClassPathXmlApplication -> ClassPathResource
    * FileSystemXmlApplicationContext -> FileSystemResource
    * WebApplicationContext -> ServletContextResource
* ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL 접두어(+ classapth:) 중 하나를 사용할 수 있다.
    * classpath:pe/oh29oh29/config.xml -> ClassPathResource
    * file:///some/resource/path/config.xml -> FileSystemResource
