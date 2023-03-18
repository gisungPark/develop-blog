# 스프링 컨테이너

## 컨테이너

컨테이너란 인스턴스의 생명주기를 관리하며, 생성된 인스턴스들에게 추가 기능을 제공하는 것을 말한다.

위의 컨테이너 개념과 마찬가지로 스프링 컨테이너는 자바 객체의 생명 주기를 관리하며, 생성된 자바 객체(Bean)들에게 추가적인 기능을 제공하는 역할을 한다.&#x20;

## 스프링 컨테이너의 종류

스프링 컨테이너에는 BeanFactory와 ApplicationContext가 있다.&#x20;

### 1\_ BeanFactory

BeanFactor는 빈을 등록, 생성, 조회 등 빈을 관리하는 역할을 한다.&#x20;

```java
@Configuration
public  class AppConfigurationn {
    @Bean
    public OrderService orderService() {
        return new OrderServiceimpl();
    }

}
```





### 2\_ ApplicationContext



## BeanFactory, ApplicationContext 비교

ApplicationContext 는 BeanFactory의 하위 클래스 개념이다. 즉 BeanFactory의 빈을 관리하는 기능을 상속, 확장한 컨테이너 이다. 단순 빈 관리 뿐만 아니라 ApplicationContext는 텍스트 메시지 관리, 파일 자월 로드, 이벤트 알람 등의 부가적인 기능을 갖고 있다. 때문에 스프링 컨테이너라고 하면 주로 이 ApplicationContext를 뜻한다.&#x20;

또한 빈 관리에서도 두 컨테이너 사이의 차이점을 보이는데 BeanFactory는 처음 getBean() 메소드가 호출된 시점에서야 빈을 생성하지만, ApplicationContext는 Context 초기화 시점에 모든 싱글톤 빈을 로드한다.





{% embed url="https://steady-coding.tistory.com/459" %}

{% embed url="https://flyburi.com/277" %}

{% embed url="https://jobjava00.github.io/language/java/framework/spring/container/" %}
