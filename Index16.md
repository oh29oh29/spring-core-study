## 스프링 AOP: 개념 소개

* Aspect-oriented Programming(AOP)은 OOP를 보완하는 수단으로, 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법.

<br>

#### AOP 주요 개념

* Aspect와 Target
    * Aspect: 모듈
    * Target: Aspect가 갖고 있는 advice 적용 대상
* Advice
    * 해야하는 일
* Join point와 Pointcut
    * Join point: advice의 합류 지점(advice 실행 시점)
    * Pointcut: 구체적인 advice 실행 시점

<br>

#### AOP 구현체

* [https://en.wikipedia.org/wiki/Aspect-oriented_programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming)
* Java
    * AspectJ
    * 스프링 AOP

<br>

#### AOP 적용 방법

* 컴파일 시점
    * .java 파일을 .class 파일로 만들 때, 바이트 코드 조작을 통해서 적용 
* 로드 타임
    * .class 파일을 로딩하는 시점에 바이트 코드 조작을 통해서 적용
* 런타임 시점
    * A라는 빈을 만들 때, A라는 타입의 프록시 빈(A 빈을 감싼 빈)을 만든다.
    * 프록시 빈에서 advice를 실행한 다음에 실제 A 빈의 메소드를 실행한다.
