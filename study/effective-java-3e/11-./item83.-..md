# item83. 지연 초기화는 신중히 사용하라.

지연 초기화(lazy initailization)는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 이 기법은 정적 필드와 인스턴스 필드 모두에 사용할 수 있다.

## 지연 초기화 특징

지연 초기화는 양날의 검이다.

지연 초기화는 인스턴스 생성 시 초기화 비용은 줄지만, 그 필드에 접근하는 비용은 커진다. 따라서 초기화가 이루어지는 비율, 실제 초기화에 드는 비용, 호출 빈도에 따라 오히려 성능 떨어뜨릴 수 있다.

<mark style="background-color:red;">**인스턴스 사용 빈도가 낮지만 초기화 비용이 크다면 지연 초기화가 제 역할을 해준다.**</mark>

## 멀티 스레드 상황에서 지연 초기화

지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면, 반드시 동기화 해야한다. 심각한 버그로 이어질 수 있기 때문이다. (item78 - 공유 중인 가변 데이터는 동기화해 사용하라)

멀티 스레드 환경에서는 지연 초기화가 더욱 까다롭다. 때문에 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다. 따라서 지연 초기화가 필요할 때까지 사용하지 말자.

### 지연 초기화 예제

다음은 인스턴스 필드의 일반적인 초기화 모습이다.

```java
private final FieldType field = computeFieldValue();
```

### 1. synchronized

지연 초기화가 **\*초기화 순환성**을 깨뜨릴 것 같으면 synchronized 접근자를 사용하자. 아래는 인스턴스 필드의 지연 초기화를 synchronized 접근자를 사용한 방식이다.

```java
private FieldType field;

private synchronized FieldType getField() {
    if(field == null) {
        field = computeFieldValue();
    }
    return filed;
}
```

**\*초기화 순환성**이란, A클래스의 생성자가 B의 인스턴스를 생성하고 -> B클래스의 생성자가 C의 인스턴스를 생성하고 -> C클래스의 생성자가 A의 인스턴스를 생성하는 경우를 뜻한다고 합니다.

### 2. \[정적 필드]지연 초기화 홀더 클래스 관용구

**성능 때문에 정적 필드를 지연 초기화해야 한다면** 지연 초기화 홀더 클래스 관용구를 사용하자. 클래스가 처음 쓰일 때 비로소 초기화된다는 특성을 이용한 관용구다.

```java
private static class FieldHoler {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서, FieldHolder클래스 초기화를 촉발한다. 이 관용구의 멋진 점은 getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다는 것이다.

### 3. \[인스턴스 필드]이중검사 관용구

성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라. 이 관용구는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.

필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언해야 한다.

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if(result != null) {    // 첫 번째 검사 (락 사용 안함)
        return result;
    }
    
    synchronized(this) {    // 두 번째 검사(락 사용)
        if (field == null) {
            field = computeFieldValue();
        }
    return filed;
}
```

#### 3.1 이중 검사 관용구의 변종 1 - 단일 검사 관용구

반복해서 초기화해도 상관없는 인스턴스 필드의 경우 두번째 검사를 생략할 수 있다. 이를 단일 검사 관용구라 한다.

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if(result == null) {    // 첫 번째 검사 (락 사용 안함)
        field = result = computeFieldValue();
    }
    return result;
}
```

#### 3.2 이중 검사 관용구의 변종 2- 짜릿한 단일검사 관용구

모든 스레드가 필드의 값을 다시 계산해도 상관없고, 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사 필드 선언에서 volatile 한정자를 없애도 된다.

이 관용구는 어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 스레드당 최대 한 번 더 이뤄질 수 있다. 보통 거의 쓰지 않는다.

```java
// 이중검사 관용구의 변종 - 짜릿한 단일검사 관용구(field의 선언에 volatile 한정자를 제거)
private FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}

```

#### &#xD;

## 핵심 정리

대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다. 성능 때문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 지연 초기화 기법을 사용하자.&#x20;

* 인스턴스 필드에는 이중 검사 관용구를 사용하자.
* 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용하자.
* 반복해 초기화해도 괜찮은 인스턴스 필드에는 단일 검사 관용구를 고려하자.

