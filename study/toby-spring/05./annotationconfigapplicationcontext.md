# AnnotationConfigApplicationContext



## AnnotationConfigApplicationContext

* ApplicationContext는 스프링 컨테이너이다.
* ApplicationContext는 인터페이스로 다양한 구현체가 있다.
* AnnotationConfigApplicationContext은 어노테이션 기반의 스프링 컨테이너이며, xml 기반의 스프링 컨테이너도 존재한다.\


```java
final AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext();
ac.register(MyConfig.class);
ac.refresh();
```

```java
@Configuration
static class MyConfig {
    @Bean
    Common common() {
        return new Common();
    }

    @Bean
    Bean1 bean1() {
        return new Bean1(common());
    }

    @Bean
    Bean2 bean2() {
        return new Bean2(common());
    }
}

static class Bean1 {
    private final Common common;

    Bean1(Common common) {
        this.common = common;
    }
}

static class Bean2 {
    private final Common common;

    Bean2(Common common) {
        this.common = common;
    }
}

static class Common {
    public Common() {
        System.out.println("create new Common");
    }
}
```
