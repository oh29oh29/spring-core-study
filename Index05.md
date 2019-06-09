## IoC 컨테이너 5부: 빈의 스코프

* 별다른 설정없이 등록하는 빈들은 기본적으로 싱글톤 scope로 등록된다.

#### Scope

* 싱글톤
* 프로토타입

```java
@Component
public class Single {

}
```
```java
@Component @Scope("prototype")
public class ProtoType {
    
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {
    
    @Autowired
    ApplicationContext applicationContext;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("Single");
        System.out.println(applicationContext.getBean(Single.class));
        System.out.println(applicationContext.getBean(Single.class));
        System.out.println(applicationContext.getBean(Single.class));

        System.out.println("ProtoType");
        System.out.println(applicationContext.getBean(ProtoType.class));
        System.out.println(applicationContext.getBean(ProtoType.class));
        System.out.println(applicationContext.getBean(ProtoType.class));
    }
}
```
출력 결과
```text
Single
pe.oh29oh29.springcoreexample.Single@10b3df93
pe.oh29oh29.springcoreexample.Single@10b3df93
pe.oh29oh29.springcoreexample.Single@10b3df93
ProtoType
pe.oh29oh29.springcoreexample.ProtoType@ea27e34
pe.oh29oh29.springcoreexample.ProtoType@33a2499c
pe.oh29oh29.springcoreexample.ProtoType@e72dba7
```

<br>

#### 프로토타입 빈이 싱글톤 빈을 참조하면?

* 프로토타입 빈에서 싱글톤 빈을 참조하면 아무 문제 없이 의도대로 동작할 것이다.

<br>

#### 싱글톤 빈이 프로토타입 빈을 참조하면?

* 싱글톤 빈에서 프로토타입 빈을 참조하게 되면 프로토타입 빈이라고 하더라도 항상 동일한 객체를 사용하게 된다.

```java
@Component
public class Single {

    @Autowired
    ProtoType protoType;

    public ProtoType getProtoType() {
        return protoType;
    }
}
```
```java
@Component @Scope("prototype")
public class ProtoType {
    
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {
    
    @Autowired
    Single single;
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(single.getProtoType());
        System.out.println(single.getProtoType());
        System.out.println(single.getProtoType());
    }
}
```
출력 결과
```text
pe.oh29oh29.springcoreexample.ProtoType@7bdf6bb7
pe.oh29oh29.springcoreexample.ProtoType@7bdf6bb7
pe.oh29oh29.springcoreexample.ProtoType@7bdf6bb7
```

위 문제를 해결하려면 즉, 싱글톤 빈에서 항상 다른 객체를 사용하려면 아래와 같은 방법을 사용하면 된다.

```java
@Component
public class Single {

    @Autowired
    ProtoType protoType;

    public ProtoType getProtoType() {
        return protoType;
    }
}
```
```java
@Component @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ProtoType {
    
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {
    
    @Autowired
    Single single;
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(single.getProtoType());
        System.out.println(single.getProtoType());
        System.out.println(single.getProtoType());
    }
}
```
출력 결과
```text
pe.oh29oh29.springcoreexample.ProtoType@df5f5c0
pe.oh29oh29.springcoreexample.ProtoType@308a6984
pe.oh29oh29.springcoreexample.ProtoType@66b7266
```

proxyMode를 ScopedProxyMode.TARGET_CLASS 로 하게 되면, 실제 인스턴스를 감싸는 프록시 인스턴스가 만들어지고 이 인스턴스가 빈으로 등록된다.

