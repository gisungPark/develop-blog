# item87. 커스텀 직렬화 형태를 고려해보라

클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면 현재 구현에 종속적이게 된다.

## 기본 직렬화

* 기본 직렬화 형태는 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아낸다.
* 심지어 이 객체들이 연결된 위상까지 기술한다. 그러나 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.

## 기본 직렬화 선택이 적합한 경우

* 유연성, 성능 ,정확성 측면에서 신중히 고민한 후 기본 직렬화 형태를 사용해야 한다.
* 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 써야 한다.
* 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.

사람의 이름을 간략히 표현한 다음 예는 기본 직렬화 형태를 써도 무방하다.

```java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 함.
     * @serial
     */
    private final Stirng lastName;

    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;

    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;

    ... // 나머지 코드는 생략
}
```

기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.

## 직렬화에 접합하지 않은 경우

다음 클래스는 직렬화 형태에 적합하지 않은 예로, 문자열 리스트를 표현하고 있다.

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    
    ... // 나머지 코드는 생략
}
```

물리적으로는 문자열들을 이중 연결 리스트로 연결했다. 이 클래스에 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 엔트리를 철두철미하게 기록한다.

객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 4가지 면에서 문제가 생긴다.

1. 공객 API가 현재 내부 표현 방식에 영구히 묶인다.
2. 너무 많은 공간을 차지할 수 있다.
3. 시간이 너무 많이 걸릴 수 있다.
4. 스택 오버플로를 일으킬 수 있다.



## StringList 를 위한 합리적인 직렬화 형태

```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    // 이번에는 직렬화 하지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // 지정한 문자열을 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * StringList 인스턴스를 직렬화한다.
     */
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeInt(size);

        // 모든 원소를 순서대로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        int numElements = stream.readInt();

        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }
    ... // 나머지 코드는 생략
}
```

단순히 리스트가 포함된 문자열의 개수를 적은 다음, 그 뒤로 문자열들을 나열하는 수준이면 될 것이다. StringList의 물리적인 상세 표현은 배제한 채 논리적인 구성만 담는 것

* transit 키워드가 붙은 필드는 기본 직렬화 형태에 포함되지 않는다.
* 모든 필드가transient 로 선언되었더라도 writeObject와 readObject 메서드는 각각 defaultWriteObject와 defaultReadObject 메서드를 호출한다.
* 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화하면 새로 추가된 필드들은 무시될 것이다.
* 구버전 defaultReadObject를 호출하지 않는다면 역직렬화할 때 StreamCorruptedException이 발생할 것이다.

### StringList 기본 직렬화, 커스텀 직렬화 성능 비교

문자열들의 길이가 평균 10이라면, 개선 버전의 StringList의 직렬화 형태는 원래 버전의 절반 정도의 공간을 차지하며, 내 컴퓨터에서는 두 배 정도 빠르게 수행된다. 마지막으로, 객선한 StringList는 스택 오버플로가 전혀 발생하지 않아 실질적으로 직렬화 할 수 있는 크기 제한이 없어진다.&#x20;

## 핵심 정리

클래스를 직렬화하기로 했다면 어떤 직렬화를 형태를 사용할지 심사숙고하기 바란다.&#x20;

자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고, 그렇지 않으면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고인하라.&#x20;

직렬화 형태도 공개 메서드를 설계할 때 준하는 시간을 들여 설계해야 한다. 한번 공개된 메서드는 향후 릴리스에서 제거할 수 없듯이, 직렬화 형태에 포함된 필드도 마음대로 제거할 수 없다. 직렬화 호환성을 유지하기 위해 영원히 지원해야 하는 것이다. 잘못된 직렬화 형태를 선택하면 해당 클래스의 복잡성과 성능에 영구히 부정적인 영향을 남긴다.



