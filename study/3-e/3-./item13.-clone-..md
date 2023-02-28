# item13. clone 재정의는 주의해서 진행해라.

## 메서드 하나 없는 Cloneable 인터페이스는 대체 무슨 일을 할까?

해당 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 <mark style="color:orange;">**CloneNotSupportedException**</mark>을 던진다.

아래 Student 클래스는 Cloneable 인터페이스를 구현하지 않아 CloneNotSupportedException 을 던진다.

```java
public class Student {
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

Student obj = new Student();
Object clone = null;
try {
    clone = obj.clone();
} catch (CloneNotSupportedException e) {
    throw new RuntimeException(e);
}
```

실무에서 Cloneable을 구현한 클래스는 clone메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 하지만 실제는 기대와 다른다. clone 메서드의 일반 규약은 허술하다. 아래에서 살펴보자.

```
이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.
일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
x.clone() != x

또한 다음 식도 참이다.
x.clone().getClass() == x.getClass()

하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.
한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.
x.clone().equals(x)

관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 모든 상위 클래스가
이 관례를 따른다면 다음 식은 참이다.
x.clone().getClass() == x.getClass()

관례상 반환된 객체와 원복 객체는 독립적이어야 한다. 이를 만족하려면 super.clone()으로 얻은 객체의
필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다. 

```



## 제대로 동작하는 Clone 메서드

### 가변 상태를 참조하지 않는 클래스의 clone 메서드

```java
@Override
protected Student clone() {
    try {
        return (Student)super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

### 가변 상태를 참고하는 클래스의 clone 메서드

앞서 간단했던 clone 메서드의 구현이 가변 객체를 참조하는 순간 재앙으로 돌변한다.

```java

public class Stack {

    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }


    @Override
    public Stack clone() {
        try {
            Stack clone = (Stack) super.clone();
            clone.elements = elements.clone();
            return clone;
        } catch (CloneNotSupportedException e) {
            new AssertionError();
        }
        return null;
    }
}

```

clone 메서드가 단순힌 super.clone()의 결과를 반환했다면 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 인스턴스와 같은 배열을 참조할 것이다. 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 이야기이다.&#x20;

Stack 클래스의 하나뿐인 생성자를 호출한다면 이러한 상황은 절대 일어나지 않는다. clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다. 그래서 stack의 clone 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해주는 것이다.&#x20;

배열의 clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 따라서 배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장한다.&#x20;

### 재귀적 호출의 한계

위에서 본 코드와 같이 clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다. 이번에는 해쉬 테이블용 clone메서드를 생각해보자&#x20;

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = new Entry[10];

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    protected Object clone() {
        try {
            HashTable clone = (HashTable) super.clone();
            clone.buckets = buckets.clone();
            return clone;
        } catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```

복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다. 이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야 한다.

```java
{
    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }


    @Override
    protected Object clone() {
        try {
            HashTable clone = (HashTable) super.clone();
            clone.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null) {
                    clone.buckets[i] = buckets[i].deepCopy();
                }
            }
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

HashTable.Entry 클래스는 깊은복사(deep copy)를 지원하도록 보강하고, clone 메서드는 각 버킷에 대해 깊은 복사를 수행한다. 연결 리스트를 재귀적으로 복사하는 이 방법은 간단하나, 연결 리스트를 복제하는 방법으로는 그다지 좋지 않다. 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있다.&#x20;

따라서 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

```java
 Entry deepCopy() {
    Entry clone = new Entry(key, value, next);
    for (Entry p = clone; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return clone;
}
```



### <mark style="color:orange;">복사 생성자와 복사 팩터리 방식의 객체 복사</mark>

복사 생성자와 그 변형인 복사 팩터리는 Cloneable/clone 방식보다 나은 면이 많다. 언어 모순적이고 위험천만한 객체 생성 메커니즘을 사용하지 않으며, 엉성화게 문서화된 규약에 기대지 않고, 정상적인 final 필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지 않고, 형변환도 필요치 않다.&#x20;

```java
public Yum (Yum yum){}
```

```java
public static Yum newInstance(Yun yum){}
```





> Cloneable이 몰고 온 모든 문제를 뒤짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다.(아이템 67) 기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는 게 최고'라는 것이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 예외라 할 수 있다.&#x20;
