# 자동 구성 애노테이션 적용

## @Configuration(proxyBeanMethods = false)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration(proxyBeanMethods = false)
public @interface MyAutoConfiguration {

}
```

