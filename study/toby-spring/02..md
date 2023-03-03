# 02. 스프링 부트 시작하기

애플리케이션을 개발할 때 http 요청과 응답 각 요소들을 어떻게 가져올 수 있고, 생성할 수 있는가 고민하면서 개발해야한다.

```java
@SpringBootApplication
public class TobyspringApplication {

    public static void main(String[] args) {
        SpringApplication.run(TobyspringApplication.class, args);
    }

}
```

위의 간단한 코드 으로 스프링에서 tomcat이 동작하고, 스프링 컨테이너도 뜬다. 컨테이너와 관련 모든 작업들이 자동화 되고, 스프링이 기동되게 만들어준다. 덕분에 우리는 스프링에 올라갈 코드만 만들면 된다.&#x20;

* tomcat은 스프링의 대표적인 서블릿 컨테이너이다.
