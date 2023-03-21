# item22. 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 달리 말해, <mark style="color:orange;">**클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다.**</mark> 인터페이스는 오직 이 용도로만 사용해야한다.&#x20;

## 상수 인터페이스(Constant Interface)

상수 인터페이스; 메서드 없이 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스



상수 인터페이스(Constant Interface)를 Implement 할 경우, 인터페이스 클래스 명을 네임스페이스로 붙이지 않고 바로 사용할 수 있다. 이러한 편리성 때문에 사용하나 해당 방법은  Anti 패턴으로 분류된다.

```java
public interface Constants {
    double PI = 3.14159;
    double PLANCK_CONSTANT = 6.62606896e-34;
}

public class Calculations implements Constants {
    public double getReducedPlanckConstant() {
        return PLANCK_CONSTANT / (2 * PI);
    }
}
```

## 상수 인터페이스의 문제점

* Implement 할 경우 사용하지 않을 수도 있는 상수 값을 계속 가지고 있어야 한다.&#x20;
* 상수 인터페이스와 Implement한 클래스에서 같이 상수를 가질 경우, 사용자가 의도와는 다르게 동작할 수 있다.(클래스 구현이 우선)

```java
public interface Constants {
    public static final int CONSTANT = 1; // *
}

public class Class1 implements Constants {

    public static final int CONSTANT = 2; // *

    public static void main(String args[]) throws Exception {
        System.out.println(CONSTANT);
    }
}
```

## 상수 공개 목적으로 올바른 케이스 3가지

* 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스 인터페이스 자체에 추가해야한다.&#x20;
* 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 된다.
* 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개하자.

{% code title="* 상수 유틸리티 클래스" %}
```java
public final class PhysicalConstants {

    private PhusicalConstants() {} // 인스턴스 방지
    
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
}

// 예제 01
public class MainClass {

    public static double method01() {
     return PhysicalConstants.AVOGADR; 
    }
}

import static PhysicalCo.AVOGADR;
public class MainClass {

    public static double method02() {
     return AVOGADR; 
    }
}
```
{% endcode %}

## 핵심정리

인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.&#x20;



