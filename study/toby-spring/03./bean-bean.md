---
description: https://www.youtube.com/watch?v=ShR5CmEUyRY
---

# 스프링 Bean이란?, Bean을 관리하는 컨테이너란?

## spring FW 등장한 이유

* EJB(Enterprise JavaBeans)
  * EJB 애플리케이션 작성을 쉽게 해준다. (미들웨어 )
  * EJB 는 선언적 프로그래밍이다.
  * 트랜잭션, 보안, 분산컴퓨팅 이런것들을 굉장히 쉽게 할 수 있다.
  * EJB를 구동시킬 수 있는 Web Application Server가 등장
  * 그러나 EJB 는 너무 어렵고, 멸시 받음

## Expert One-on-One :: J2EE Design and  Development

* 로드 존슨
  * 스프링을 소개한다.
  * EJB에서 했던 일들을 Java Bean이 처리할 수 있게 된다.&#x20;

## Bean이란?

*   java에서 인스턴스 생성

    * Book book = new Book();


* <mark style="background-color:yellow;">**Bean은 컨테이가 관리하는 객체를 말한다.**</mark>&#x20;
* 객체의 생명주기를 컨테이너가 관리한다.
* <mark style="background-color:yellow;">**객체를 싱글턴으로 만들 것인지, 프로토타입으로 만들것인지**</mark>
  * 스프링은 매번 bean을 생성하는 프로토타입도 지원한다.
* 개발자는 인스턴스를 직접 생성할 것이 아니라, 컨테이너에 의해 관리되도록 해야한다.&#x20;

## 스프링의 핵심 기능

* 관점 지향 컨테이너
  * 빈을 생성, 관리한다.
  * 관점지향(AOP, aspect-oriented programming)

&#x20;&#x20;

## Book 클래스의 인스턴스 생성&#x20;

```
Book book1 = new Book("즐거운 자바", 2000);
```

1\) new Book("즐거운 자바", 2000);

* 생성자가 호출되면 Heap 메모리에 인스턴스가 생성된다.&#x20;

2\) book1은 1)에서  생성한 인스턴스를 참조한다.&#x20;

* book1 은 참조변수



## 스프링 컨테이너

id 값을 넘겨주면 id에 해당하는 bean 객체를 생성하여 전달해준다. \




## 스프링이란? 엄청 잘 만든 applicationContext를 제공해준다.

* 객체를 싱글턴으로 생성해주고, 자기 자신도 싱글턴인 ApplicationContext
  * 컨테이너 역할을 수행한다.&#x20;

```java
public class ApplicationContext {

    Properties props;
    private Map<String, Object> objectMap = new HashMap<>();

    // 1. 싱글턴 패턴 적용 - 자기 자신을 참조하는 static 필드를 선언한다.
    private static ApplicationContext applicationContext = new ApplicationContext();

    // 3. 싱글턴 패턴 적용 - 인스턴스 반환 static 메서드 생성
    public static ApplicationContext getInstance(){
        return applicationContext;
    }

    // 2. 싱글턴 패턴 적용 - 생성자를 private으로 전환
    private ApplicationContext() {
        props = new Properties();
        try {
            props.load(new FileInputStream("src/main/resources/Beans.properties"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public Object getBean(String id) throws ClassNotFoundException, InstantiationException, IllegalAccessException {

        Object o1 = objectMap.get(id);
        if(o1 != null) return o1;

        String className = props.getProperty(id);
        Class clazz = Class.forName(className);
        Object o = clazz.newInstance();

        objectMap.put(id, o);

        return objectMap.get(id);
    }

}
```

\


\


