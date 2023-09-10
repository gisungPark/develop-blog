---
description: 이 장에서는 람다가 탄생한 배경과 자바 8에서 광범위하게 사용된 소프트웨어 개발 패턴인 동작 파라미터화를 설명한다.
---

# 동작 파라미터화 코드 전달하기

변화하는 요구사항은 소프트웨어 엔지니어링에서 피할 수 없는 문제다. 동작 파라미터화를 이용하면 자주 바뀌는 요구사하에 효과적으로 대응할 수 있다. 동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다. 이 코드 블록의 실행은 프로그램에서 호출 시점까지 미뤄진다.&#x20;



## 변화하는 요구사항에 대응하기

### 기존 농작 재고목록에서 녹색 사과만 필터링하는 기능 코드

```java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if(GREEN.equals(apple.getColor()) {
            result.add(apple);
        }
    }
}

```

### 색을 파라미터화

색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항(빨강, 초록)에 좀 더 유연하게 대응하는 코드를 만들 수 있다.

```java

public static List<Apple> filterGreenApples(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if(color.equals(apple.getColor()) {
            result.add(apple);
        }
    }
}
```

### 가능한 모든 속성으로 필터링

위 코드를 본 농부가 다시 나타나서는 '색 이외에도 가벼운 사과와 무거운 사과로 구분할 수 있게 해주세요'라고 요구한다. 이렇듯 요구사항이 앞으로도 바뀔 여지가 있음을 알게되었다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color,
                                                    int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if( (flag && apple.getColor.equals(color)) 
                || (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }
}



filterApples(inventory, GREEN, 0, true);
filterApples(inventory, null, 150, false);
```

위 코드는 모든 속성을 메서드 파라미터로 추가한 모습으로 형편없는 코드다. true, false는 뭘 의미하는지 이해하기 어렵고 요구사항 변경에 유연하게 대응할 수도 없다.&#x20;

## 동작 파라미터화

앞 절에서 파라미터를 추가하는 방법이 아닌 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 절실하다는 것을 확인했다. 우리의 선택 조건을 다음처럼 결정할 수 있다. 사과의 어떤 속성에 기초해서 불리언값을 반환하는 방법이 있다. (참 또는 거짓을 반환하는 함수를 프레디케이트라고한다.) 선택 조건을 결정하는 인터페이스를 정의하자.

동작 파라미터화, 즉 메서드가 다양한 동작을 받아서 내부적으로 다양한 동작을 수행할 수 있다.&#x20;

### 추상적 조건으로 필터링

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
} 


Public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {

    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if(p.test(apple)) {
            result.add(apple);
        }
    }
}
```

위 코드는 우리가 전달한 ApplePredicate 객체에 의해 filterApples 메서드의 동작이 결정되는 유연성을 가지게 되었다. 컬랙션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동적 파라미터화의 강점이다.&#x20;



<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

### 익명 클래스 사용

위 예제에서 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의하고 인스턴스하는 작업은 번거로운 작업이다.  이를 위해 자바에서는 클래스 선언과 인스턴스화를 동시에 수행할 수 있도록 익명 클래스라는 기법을 제공한다.&#x20;

```java
List<Apple> reaApples = filterApples(inventory, new ApplePredicate() { 
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor();
    }
});
```

### 람다 표현식 사용

익명 클래스로도 아직 부족한 점이 있다. 먼저 익명 클래스는 반복되어 지저분한 코드로 많은 공간을 차지한다. 둘째, 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다. (\*page80 - 퀴즈 2.2.참조)

자바 8의 람다 표현식을 사용하면 위 예제를 다음처럼 간단하게 재구현할 수 있다.

```java
List<Apple> result = filterApples(inventory, 
                                (Apple apple) -> RED.equals(apple.getColor()));
```

### 리스트 형식으로 추상화

다음과 같이 형식 파라미터 T를 통해서 바나나, 오렌지, 정수, 문자열들의 리스트까지 확장해서 사용가능해 졌다.&#x20;

```java
public interface Predicate<T> {
    public boolean test(T t);
} 


Public static <T> List<T> filter(List<T> inventory, Predicate<T> p) {

    List<T> result = new ArrayList<>();
    for(T e : inventory) {
        if(p.test(e)) {
            result.add(e);
        }
    }
}
```



## 요약 정리

지금까지 동작 파라미터화가 변화하는 요구사항에 쉽게 적응하는 유용한 패턴임을 확인했다. 동작 파라미터화 패턴은 동작을 캡슐화한 다음에 메서드로 전달해서 메서드의 동작을 파라미터화한다.&#x20;

