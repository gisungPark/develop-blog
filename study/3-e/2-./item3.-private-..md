# item3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 싱글턴은 클래스를 싱글턴으로 만들면 이를 사용하는 클라이어트를 테스트 하기 어렵다는 단점이 있다.

## 싱글턴 생성을 위한 2가지 방법

* public static 멤버가 final 필드 방식

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();

	private Elvis();
}
```

해당 방식의 첫 번째 장점은

* 정적 팩터리 메서드를 public static 멤버 제공 방식

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();

	private Elvis();

	public static Elvis getInstance() {
		return INSTANCE;
	}
}
```

해당 방식의 장점은

첫째. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다

둘째. 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다.

셋째. 정적 팩터리의 메서 참조를 공급자(supplier)로 사용 할 수 있다. (Elvis::getInstance)

```java
💡싱글턴 클래스를 직렬화하는 방법
싱글턴 클래스를 직려화하기 위해선 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다.
모든 인스턴스 필드를 일시적(transient)라고 선언하고 readResolve 메서드를 제공해야 한다.
이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

private Object readResolve() {
	return INSTANCE;
}
```

* Enum 클래스 활용 방식

```java
public enum Elivs {
	INSTANCE;

	public void leaveTheBuilding() {}
}
```

public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력없이 직렬화할 수 있는 방법이다. 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다. **대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.** 단, 만들려는 싱글턴이 Enum 외에 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
