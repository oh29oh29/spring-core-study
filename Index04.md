## IoC 컨테이너 4부: @Component와 컴포넌트 스캔

#### @Component

* @Repository
* @Service
* @Controller
* @Configuration

<br>

#### @ComponentScan

* 스프링 3.1부터 도입
* basePackages vs basePackagesClasses
    * 지정된 클래스를 기준으로 컴포넌트를 스캔한다.
    * basePackages는 type-safe 하지 않음.
    * basePackagesClasses는 type-safe 한 방법(recommended).

<br>

#### 컴포넌트 스캔 주요 기능

* 스캔 위치 설정
* 필터: 어떤 annotation을 스캔할지 안할지 정의

<br>

#### 동작 원리

* 실제 스캐닝은 [ConfigurationClassPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)라는 [BeanFactoryPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)에 의해 처리된다.

