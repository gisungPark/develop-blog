# item20. 추상 클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스, 이렇게 두 가지 방안이 있다. 둘의 가장 큰 차이는 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. (자바는 단일 상속만 지원하니, 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약을 안는 반면, 인터페이스는 여러 클래스 타입을 가질 수 있다.)

## 인터페이스

인터페이스(interface)는 [자바 프로그래밍 언어](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94\_\(%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D\_%EC%96%B8%EC%96%B4\))에서 [클래스](https://ko.wikipedia.org/wiki/%ED%81%B4%EB%9E%98%EC%8A%A4\_\(%EC%BB%B4%ED%93%A8%ED%84%B0\_%EA%B3%BC%ED%95%99\))들이 구현해야 하는 동작을 지정하는데 사용되는 [추상 자료형](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81\_%EC%9E%90%EB%A3%8C%ED%98%95)이다.

### 장점

* 인터페이스도 디폴트 메서드를 제공할 수 있다.(자바 8 이후)
  * 인터페이스의 메서드 중 구현이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해 편의를 높일 수 있다.
  * 그러나 디폴트 메서드에서 인스턴스 필드를 사용해야하는 경우, 추상클래스를 쓸 수 밖에 없다.&#x20;
* 인터페이스 방식은 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.
* 믹스인(mixin) 정의에 안성맞춤이다.&#x20;
  * 믹시인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
* 계층구조가 없는 타입 프레임워크를 만들 수 있다.
  * 계층구조가 명확하지 않은 경우, 인터페이스를 조합해서 새로운 클래스를 생성할 수 있다.

```java
public interface Singer {
    AudioClip sing(Song s);
}
public interface SongWriter {
    Song compose(int chartPosition);
}

public interface SingerSongWriter extends Singer, Songwriter {
    AudioClip sing(Song s);
    Song compose(int chartPosition);
}
```

### 단점

* 인터페이스는 인스턴스 필드를 가질 수 없고, public이 아닌 정적 멤버도 가질 수 없다.(private 정적 메서드는 가질  수 있다.)
* equals와 hashCode 같은 Object의 메서드는 디폴트 메서드로 제공할 수 없고, 제공 해서도 안된다.(컴파일 에러 발생)

## 추상클래스

자바에서는 하나 이상의 추상 메소드를 포함하는 클래스를 가리켜 추상 클래스(abstract class)라고 한다.\
이러한 추상 클래스는 객체 지향 프로그래밍에서 중요한 특징인 다형성을 가지는 메소드의 집합을 정의할 수 있도록 해준다.

즉, 반드시 사용되어야 하는 메소드를 추상 클래스에 추상 메소드로 선언해 놓으면, 이 클래스를 상속받는 모든 클래스에서는 이 추상 메소드를 반드시 재정의해야 한다.

* 인터페이스와는 달리 기존 클래스에 추상 클래스를 끼워넣기는 어렵다.



## 인터페이스와 추상 골격 클래스

* 인터페이스와 추상 클래스의 장점을 모두 취할 수 있다.
  * 인터페이스 - 디폴트 메서드 구현
  * 추상 골격 클래스 - 나머지 메서드 구현
  * 템플릿 메서드 패턴
* 다중 상속을 시뮬레이트할 수 있다.
* 골격 구현은 상속용 클래스이기 때문에 아이템 19를 따라야 한다.

```java
static List<Integer> intArrayAsListByAbstract(int[] a){

        return new AbstractList<Integer>() {
            @Override
            public Integer get(int index) {
                return null;
            }

            @Override
            public int size() {
                return 0;
            }
        };
    }

```

골격 구현 클래스의 아름다움은 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다는 점에 있다.\


<figure><img src="../../../../.gitbook/assets/image (1) (1).png" alt="일반 인터페이스를 구현하는 경우와 추상 골격 클래스를 구현하는 케이스"><figcaption><p>일반 인터페이스를 구현하는 경우와 추상 골격 클래스를 구현하는 케이스</p></figcaption></figure>



## 핵심정리

일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.&#x20;




