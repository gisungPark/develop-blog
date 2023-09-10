# 메타 애노테이션과 합성 애노테이션

## 강의 목적

@AutoConfiguration을 잘 이해하려면, 스프링 부트가 애노테이션을 활용할 때 사용하는 몇가지 기법을 이해하고 코드를 살펴볼 수 있어야 하며 우리도 그걸 응용하는 지식을 쌓자.



<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-19 오후 3.28.29.png" alt=""><figcaption></figcaption></figure>

## 메타 애노테이션

Stereo Type(스테레오 타입) 애노테이션들은 @Component 애노테이션을 자신의 애노테이션으로 다시 부여해서 @Component 애노테이션과 동일한 효과를 낼 수 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Component
public @interface MyComponent {
}

```



### 메타 애노테이션의 장점

* @Component를 직접 붙이는 것과 @Component를 메타 애노테이션으로 가진, @Service, @Controller 애노테이션을 붙이는 것에는 기능적 차이가 없다.
* 그러나 이름으로부터 얻는 명시적 역할 분석이 가능하며,  추가 기능 제공이 가능하다.&#x20;



주의,

메타 애노테이션과 상속은 다른 개념이다.&#x20;

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@UnitTest
@interface FastUnitTest {

}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE, ElementType.METHOD})
@Test
@interface UnitTest {
}

```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 합성 애노테이션

메타 애노테이션을 하나 이상을 조합해서 사용하는 경우, 합성 애노테이션을 사용하면 하나의 애노테이션으로 포함한 모든 애노테이션이 다 적용된 것 같은 효과를 낸다.&#x20;

