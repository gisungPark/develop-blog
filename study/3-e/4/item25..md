# item25. 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스를 여러개 선언하더라도 자바 컴파일러는 불평하지 않는다. 하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야하는 행위다. 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문이다.&#x20;

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
