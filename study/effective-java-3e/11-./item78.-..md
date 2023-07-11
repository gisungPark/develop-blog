# item78. 공유 중인 가변 데이터는 동기화해 사용하라.

## 동기화란?

대다수가 동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다. 그러나 동기화는 2가지 프로세스로 나눠 생각할 수 있다.

* 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다. 이 과정 중에 다른 관섭은 불가하다.
* 동기화된 메서드나 블록에 들어간 스레드가 같은 락 아래에 수행된 모든 수정의 최종 결과를 보게해 준다.

아래 예시를 살펴보자.

언어 명세상 long과 double 외의 변수를 읽고 쓰는 동작은 원자적이다. 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻이다.

"이를 읽고 원자적 데이터를 읽고 쓸때는 동기화하지 말아야겠다"고 생각한다면 기대와 다른 결과를 마주할 수 있다. 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장하지만, **한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다.** 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.

공유 중인 가변 데이터를 비록 원자적으로 읽고 쓸 수 있을지라도 동기화에 실패하면 처참한 결과로 이어질 수 있다.

{% code lineNumbers="true" %}
```java
public class 동기화예제 {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        final Thread backGroudThread = new Thread(() -> {
            int i = 0;
            *while (!stopRequested) {
                i++;
            }
        });
        backGroudThread.start();
        TimeUnit.SECONDS.sleep(1);
        *stopRequested = true;
    }
}

```
{% endcode %}

위 예제는 1초뒤 종료될 것을 기대했지만, 종료되지 않는다. 물론 원인은 동기화에 있다. **동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보장할 수 없다.**

## synchronized

{% code lineNumbers="true" %}
```java
public class 동기화예제02 {
    private static boolean stopRequested;

    *private static synchronized void requestStop() {
        stopRequested = true;
    }
    *private static synchronized boolean getStropRequest() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        final Thread backGroudThread = new Thread(() -> {
            int i = 0;
            *while (!getStropRequest()) {
                i++;
            }
        });
        backGroudThread.start();
        TimeUnit.SECONDS.sleep(1);
        *requestStop();
    }
}
```
{% endcode %}

&#x20;synchronized 키워드를 통해 공유 변수의 동기화된 값을 얻을 수 있다. 다음과 같이 코드를 수정하면 기대한 1초뒤 종료하는 결과를 얻을 수 있다. **이때 읽기와 쓰기 메서드 모두를 동기화했음에 주목하자**. 읽기와 쓰기 모두가 동기화되지 않으면 동작을 보장하지 않는다.

## Volatile

volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

{% code lineNumbers="true" %}
```java
public class 동기화예제03 {
    *private static volatile boolean stopRequested = false;

    public static void main(String[] args) throws InterruptedException {
        final Thread backGroudThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        backGroudThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}

```
{% endcode %}

volatile 키워드는 주의해서 사용해야 한다. 예를 들어 다음은 일련번호를 생성할 의도록 작성한 메서드이다.

{% code lineNumbers="true" %}
```java
public class volatile_예제 {
    private static volatile int nextSerialNumber = 0;

    public static int generateSerialNumber() {
        *return nextSerialNumber++;
    }
}
```
{% endcode %}

이 메서드는 매번 고유한 값을 반환할 의도록 만들어졌다. 하지만 이 역시 동기화 없이는 올바르게 동작하지 않는다. 문제는 증가 연산자(++)이다. 이 연산자는 코드상으로는 한줄이지만 실제로는 nextSerialNumber 필드에 두 번 접근한다.

1. 값을 읽는다.
2. 증가한 새로운 값을 저장한다.

만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 된다. generateSerialNumber 메서드에  synchronized 한정자를 붙이면 이 문제가 해결된다. 동시에 호출해도 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다. (단, 메서드에 synchronized를 붙였다면  volatile은 제거해야 한다.)

배타적 실행은 필요 없고 스레드끼리의 통신만 필요하다면 volatile 한정자만으로 동기화할 수 있다. 다만 올바로 사용하기가 까다롭다.



## 핵심요약

가변 데이터든 단일 스레드에서만 쓰도록 하자. 멀티 스레드는 디버깅 난이도가 높다. 특정 타이밍에만 발생할 수도 있고, vm에 따라 현상이 달라지기도 한다.

여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야한다.동기화하지 않으면 한 스레드가 수정한 내용을 다른 스레드가 보지 못할 수도 있다.
