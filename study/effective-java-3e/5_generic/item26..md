# item26. 로 타입은 사용하지 말라

## 제네릭이란

클래스와 인터페이스 선언에 타입 매개변수가 쓰이면, 이를 제네릭 클래스 혹은 제네릭 인터페이스라 한다. 제네릭 클래스와 제네릭 인터페이스를 통틀어 제네릭 타입(generic type)이라 한다.

제네릭 타입에서 타입 매개변수를 사용하지 않은 경우를 <mark style="color:orange;">**로 타입**</mark>이라 하는데, 이는 권장되는 사용법이 아니다. 제네릭이 사용되기 전 코드와 호환되도록 하기 위한 궁여지책에 불과하다.

## 로 타입의 예외 발생 가능성

* 로 타입이 원인이 되는 예외 케이스\
  stamp 대신 coin을 넣어도 아무 오류 없이 컴파일되고 실행된다.&#x20;

```java
private final Collection stamps = ...;

stamps.add(new Coin());

for(Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp)i.next();
}
```

오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할 때 발결하는 것이 좋다. 위 예제에서는 런타임에서야 문제를 알아챌 수 있다. <mark style="color:orange;">**제네릭을 활용하면 이 정보가 주석이 아닌 타입 선언 자체에 녹아든다.**</mark> 컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에 보이지 않는 형변환을 추가하여 절대 실패하지 않음을 보장한다.&#x20;

로 타입(타입 매개변수가 없는 제네릭 타입)을 쓰는 걸 언어 차원에서 막아 놓지는 않았지만 절대로 써서는 안 된다. 로 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 읽게 된다.&#x20;

## List(로 타입)과 List\<Object> 비교

아래 코드는 컴파일되나 strings.get(0)을 형변환하려 할 때, ClassCastException을 던진다. Integer 값을 String으로 형 변환하려 했기 때문이다.

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<String>();
    unsafeAdd(strings, Integer.valueOf(42));
    
    String s = strings.get(0);
    System.out.println(s);
}
private static void unsafeAdd(List list, Object o) {
    list.add(o);
}
```

아래는 List\<String> 객체를 List\<Object>의 파라미터로 전달하는 예제이다. 해당 코드는 List\<String> 타입과List\<Object> 타입이 일치하지 않는다며, 컴파일 조차 되지 않는다.&#x20;

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<String>();
    unsafeAdd(strings, Integer.valueOf(42));
    
    String s = strings.get(0);
    System.out.println(s);
}
private static void unsafeAdd(List<Object> list, Object o) {
    list.add(o);
}
```





## 핵심정리

로 타입을 사용하면 런타입에 예외가 일어날 수 있으니 사용하면 안 된다. 로 타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다.&#x20;

* Set\<Object>; 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입
* Set\<?> 모종의 타입 객체만 저장할 수 있는 와일드카드 타입

제테릭 타입인 Set\<Object>와 Set\<?> 은 안전하지만, 로 타입인 Set은 안전하지 않다.
