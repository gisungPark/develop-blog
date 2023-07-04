# item73. 추상화 수준에 맞는 예외를 던지라.

메서드가 저수준 예외를 처리하지 않고 바깥으로 전파(throws) 하는 경우에 대한 두 가지 문제점

* 수행하려던 일과 관련없어 보이는 예외가 발생할 수 있다.
* 구현 방식을 바꾸면 또 다른 예외를 발생시켜 기존 클라이언트 프로그램을 깨지게 할 수 있다.

이 문제를 피하려면 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다. 이를 예외번역이라 한다.



```java
try {
    // 저수준 추상화를 이용한다.
}catch(LowerLevelException e) {
    throw new HigherLevelException(....);
}
```

위 문제의 **해결책**

* 상위 계층에서는 **저수준 예외**를 잡아 자신의 **추상화 수준에 맞는 예외**로 바꿔 던져야 한다.
* 이를 **예외 번역(Exception Translation)** 이라 한다.



```java
public abstract class AbstractSequentialList extends AbstractList {
/**
 * Returns the element at the specified position in this list.
 *
 * @param index index of the element to return
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         (index < 0 || index >= size())
 */
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: " + index);
    }
 }
}
```

}

* **예외 번역**시 저수준 예외가 디버깅에 도움이 된다면 **예외 연쇄(exception chaining)** 를 사용하는 게 좋다.

## 예외 연쇄

* **예외 연쇄란?**
  * 문제의 **근본 원인(cause)** 인 **저수준 예외**를 **고수준 예외**에 실어 보내는 방식이다.
  * 이를 통해 별도의 **접근자 메서드(Throwable의 getCause 메서드)** 를 통해 필요하면 언제든 **저수준 예외**를 꺼내 볼 수 있다.

## 핵심 정리

아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 **예외 번역**을 사용하라. 이때 예외 연쇄를 이용하면 **상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어** 오류를 분석하기에 좋다.
