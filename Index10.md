## IoC 컨테이너 9부: ResourceLoader

* 리소스를 읽어오는 기능을 제공하는 인터페이스
* ApplicationContext extends ResourceLoader

<br>

#### 리소스 읽어오기

* 파일 시스템에서 읽어오기
* URL로 읽어오기
* 상대/절대 경로로 읽어오기
* 클래스패스에서 읽어오기
    * classpath라는 접두어를 지정해서 getResource를 하면 빌드된 classes(root)를 기준으로 리소스를 가져온다.
    
    ![classpath](/images/index10-resource-classpath.png)
    
    ```java
    @Component
    public class AppRunner implements ApplicationRunner {
    
        @Autowired
        ResourceLoader resourceLoader;
    
        @Override
        public void run(ApplicationArguments args) throws Exception {
            Resource resource = resourceLoader.getResource("classpath:/test.txt");
            System.out.println(resource.exists());
        }
    }
    ```
    출력 결과
    ```text
    true
    ```




