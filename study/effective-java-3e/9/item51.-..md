# item51. 메서드 시그니처는 신중히 설계하라.

## 메서드 이름을 신중히 짓자.

메서드의 표준 명명 규칙을 따르자. 이해할 수 있고, 같은 패키지에 속한 다름 이름들과 일관되게 짓는게 최우선 목표이다. 그다음 목표는 개발자 커뮤니티에서 널리 받아들여지는 이름을 사용하는 것이다.

## 편의 메서드를 너무 많이 만들지 말자.

메서드가 너무 많은 클래스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 어렵다. 인터페이스도 마찬가지다. 메서드가 너무 많으면 이를 구현하는 사람과 사용하는 사람 모두를 고통스럽게 한다. 확신이 서지 않으면 만들지 말자.

## 매개변수 목록은 짧게 유지하자.

매개변수는 4개 이하가 좋다. 4개가 넘어가면 매개변수를 전부 기억하기가 쉽지 않다. 여러분이 만든 API.에 이 제한을 넘는 메서드가 많다면 프로그래머들은 API문서를 옆에 끼고 개발해야 할 것이다.&#x20;

같은 타입의 매개변수 여러 개가 연달아 나오는 경우가 특히 해롭다. 사용자가 매개변수 순서를 기억하기 어려울뿐더러, 실수로 순서를 바꿔 입력해도 그대로 컴파일되고 실행 된다. 단지 의도와 다르게 동작할 뿐이다.

```java
public class EffectiveMain {

    public static void main(String[] args) {
        new Lecture("이펙티브자바", 2, 50);    // 강의명, 대상 학년, 수강 정원
        new Lecture("이펙티브자바", 50, 2);
    }

    class Lecture {
        String name;
        int schoolYear;
        int countOfStudent;

        public Lecture(String name, int schoolYear, int countOfStudent) {
            this.name = name;
            this.schoolYear = schoolYear;
            this.countOfStudent = countOfStudent;
        }
    }
}
```

매개변수 목록을 짧게 줄여주는 세가지 기술

* 여러 메서드로 쪼갠다.
* 매개변수 여러개를 묶어주는 도우미 클래스를 만들자.
* 메서드 호출에 빌더 패턴을 응용하자

### 여러 메서드로 쪼갠다.

지정된 범위의 부분 리스트에서 주어진 원소를 찾아야 하는 요구사항에 아래와 같이 설계했다고 가정하자. 해당 메서드는 부분집합의 시작점, 끝점, 찾아야할 요소를 전달한다.

```java
findElementAtSubList(int fromIndexOfSubList, int toIndexOfSubList, Object element);
```

지정된 범위의 부분 리스트에서 주어진 원소를 찾아야 하는 요구사항을 아래와 같이 두 가지로 분리를 고민해 보자.&#x20;

```java
List<E> subList(int fromIndex, int toIndex);

int indexOf(Object o);
```

&#x20;**지정된 범위의 부분 리스트를 구하는 기능**과 **주어진 원소를 찾는 기능** 이 두 기능을 생각해보면 공통점이 없다.   공통점이 없는데 하나의 메서드로 쓰이는 게 맞을까? \
이렇게 분리해서 기능으로 만든다면 다른 곳에서도 쉽게 조합해서 사용할 수 있습니다.

기능을 쪼개다보면 자연스럽게 중복이 줄고 결합성이 낮아진다. 코드를 수정하고 테스트하기 쉬워진다는 말이다.

### 매개변수를 여러를 묶어주는 도우미 클래스를 만들자.

매개변수가 많다면 매개변수를 묶어줄 수 있는 클래스를 만들어서 하나의 객체로 전달할 수 있다.

예를 들어, 카드 게임인 블랙잭을 구현한다 할때,  카드 게임에서 각 사용자에게 카드를 나누는 행위(dealing)를 구현한다면 사용자에게 카드를 나눠야 하니 사용자 이름과 카드가 필요하다.

```java
// rank는 카드의 숫자이며 suit은 카드의 무늬입니다.
dealing(String gamerName, String rank, String suit)
```

카드라는 특징을 생각했을때,  rank(카드의 숫자), suit(카드의 무늬)는 항상 같이 다니게 된다. 그렇다면 이처럼 rank와 suit을 묶는 도우미 클래스(정적 멤버 클래스)를 만들면 하나의 매개변수로 주고 받을 수 있다.

```java
dealing(String gamerName, Card card)
```

```java
class Blackjack {
    // 도우미 클래스 (정적 멤버 클래스)
    static class Card {
        private String rank;
        private String suit;
    }

    public void dealing(gamerName, card);
}
```



### 매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다.

매개변수로 적합한 인터페이스가 있다면 그 인터페이스를 직접 사용하자. 예를 들어 메서드에 HashMap 대신 Map을 사용하자. 그러면 hashMap 뿐 아니라 TreeMap, ConcurrentHashMap, TreeMap 등 어떤 Map 구현체도 인수로 건넬 수 잇다.&#x20;





