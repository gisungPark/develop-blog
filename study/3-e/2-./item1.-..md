# item1. 생성자 대신 정적 팩터리 메서드를 고려하라.

## **정적 팩터리 메서드가 생성자보다 좋은 5가지 이유**

* **이름을 가짐으로써, 반환될 객체의 특성을 쉽게 묘사할 수 있다.**

```jsx
생성자인 BigInteger(int, int, Random)과 정적 팩토리 메서드인 BigInteger.probablePrime 중
어느 쪽이 '값이 소수인 Bignteger를 반환한다'는 의마를 더 잘 설명할지 생각해 보자.
```

정적 팩터리 메서드는 개수의 제약이 없다. 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주자.

* **호출될 때마다 인스턴스를 새로 생성하지 않을 수 있다.**

new 키워드를 사용하면, 객체는 무조건 새로 생성된다. 만약 자주 생성될 것 같은 인스턴스는 클래스 내부에 미리 생성해 놓은 다음 반환한다면 코드를 최적화 할 수 있을 것이다. 아래는 `java.lang.Integer` 클래스의 `valueOf` 구현을 살펴보자.

```java
public static Integer valueOf(int i) {
	if( i > IntegerCache.low && i <= IntegerCache.high)
		return IntegerCache.cache[i + (-IntegerCache.low)];
	return new Integer(i);
}
```

전달된 i가 캐싱된 숫자 범위 내에 있다면, 객체를 새로 생성하지 않고 ‘미리 생성된’ 객체를 반환한다. 그렇지 않을 경우에만 new 키워드를 사용하여 객체를 생성한다. 이러한 방법은 생성 비용이 큰 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.

이런 클래스를 인스턴스 통제(instance-controlled) 클래스라 한다. 인스턴스를 통제하면, 클래스를 `싱글턴` 으로 만들 수도, `인스턴스화 불가` 로 만들 수도 있다. 또한 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.

```java
Q. "언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다"의 의미는?
```

* **반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**

생성자를 사용하면 생성되는 객체의 클래스가 하나로 고정된다. 그러나 정적 팩터리 메서드를 사용하면, 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 가질 수 있다.

```java
class Laptop {
	public static Laptop lowQualityLaptop() {
		return new LowQualityLaptop();
	}

	public static Laptop normalLaptop() {
		return new NormalLaptop();
	}

	public static Laptop highEndLaptop() {
		return new HighEndLaptop();
	}
}
```

이런 유연성을 응용하면, 구현 클래스를 공개하지 않고 객체를 반환 할 수 있다. 이런 아이디어는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심이다.

* 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
* **정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**

***

## **정적 팩터리 메서드가 생성자보다 안 좋은 2가지 이유**

* 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
* 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

```jsx
정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고
사용하는 것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 
무작정 public 생성자를 제공하던 습관이 있다면 고치자. 
```

* 참고 자료

[정적 메소드는 언제 써야하는가? :: 마이구미](https://mygumi.tistory.com/253)

[생성자 대신 정적 팩터리 메서드를 고려하라](https://hudi.blog/effective-java-static-factory-method/)
