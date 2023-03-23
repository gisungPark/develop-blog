# item25. 톱레벨 클래스는 한 파일에 하나만 담으라

자바에서는 하나의 소스 파일에 여러개의 클래스를 정의할 수 있다. 하지만 아무런 득이 없으며, 심각한 위험을 감수해야하는 행위이다.&#x20;

아래와 같이 한 클래스에 여러 클래스를 정의할 수 있으며, 그중 어느 것을 사용할지는 <mark style="color:orange;">**어느 소스 파일을 먼저 컴파일하냐에 따라 결과가 달라져 개발자가 의도치 않게 동작할 위험이 있다.**</mark>&#x20;

```java
 public static void main(String[] args) {
        System.out.println(Utensil.NAME + " " + Dessert.NAME);
    }
```

{% code title="Dessert.java 파일" %}
```java
// 톱 클래스 2개를 하나의 파일에서 정의
class Utensil {
    static final String NAME = "POT";
}

class Dessert {

    static final String NAME = "PIE";
}
```
{% endcode %}

{% code title="Utensil.java 파일" %}
```java
class Utensil {
    static final String NAME = "PAN";
}

class Dessert {

    static final String NAME = "CAKE";
}
```
{% endcode %}



{% code title="* 컴파일 순서에 따른 결과 변동" %}
```bash
 mn4qyt6c7r@baggiseong-ui-MacBookPro   javac Main.java
 mn4qyt6c7r@baggiseong-ui-MacBookPro  java Main
PAN CAKE
 mn4qyt6c7r@baggiseong-ui-MacBookPro   javac Main.java Dessert.java
 mn4qyt6c7r@baggiseong-ui-MacBookPro   java Main
POT PIE
 mn4qyt6c7r@baggiseong-ui-MacBookPro   javac Main.java Utensil.java
 mn4qyt6c7r@baggiseong-ui-MacBookPro   java Main
PAN CAKE
```
{% endcode %}

위 문제의 해결책은 간단하다. 톰레벨 클래스들(Utensil과 Dessert)를 서로 다른 소스 파일로 분리하면 된다. 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법을 고민해 볼 수 있다.&#x20;

아래는 앞의 예제를 정적 멤버 클래스로 바꿔본 예제이다.&#x20;

```java
public class Main02 {
    public static void main(String[] args) {
        System.out.println(Utensil.name + " " + Dessert.name);
    }
    
    private static class Utensil {
        static final String name = "pan";
    }
    
    private static class Dessert {
        static final String name = "cake";
    }
}
```



## 참고 내용

인터페이스도 클래스와 동일하게 한 파일 내, 두 개의 인터페이스 선언이 가능하다. 그러나 이 역시도 권장되지 않는다.&#x20;

```java
interface MultiInter01 {
    public static final int CONST = 1;
}

interface MultiInter02 {
    public static final int CONST = 2;
}

```

```java
public class MultiInterMain {
    public static void main(String[] args) {
        Instance01 i01 = new Instance01();
        Instance02 i02 = new Instance02();
        System.out.println(i01.CONST +" " + i02.CONST);
    }
}


class Instance01 implements MultiInter01 {
}


class Instance02 implements MultiInter02 {
}
```



## 핵심 정리

소스 파일 하나에는 반드시 톱레벨 클래스를 하나만 담자. 이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어 내는 일은 사라진다.&#x20;
