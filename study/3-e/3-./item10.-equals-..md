# item10. equals는 일반 규약을 지켜 재정의하라.

## equals를 재정의 하지 않는게 최선인 경우

* 각 인스턴스가 본질적으로 고유한 경우
  * Thread는 값이 아닌 동작 개체를 표현하는 클래스이기 때문에 동일한 인스턴스가 애초에 없다.
* 인스턴스의 ‘논리적 동치성’을 검사할 일이 없는 경우
*   상위 클래스에서 재정의한 equals가 하위 클래스에도 적용되는 경우

    (ex. Set의 구현체는 AbstractSet이 구현한 equals를 상속받아 사용한다.)
*   클래스가 private이거나 package-private 이고, equals 메서드를 호출할 일이 없는 경우

    이 경우 equals가 실수로라도 호출되는 걸 막고싶다면 아래와 같이 재정의하면 된다.&#x20;

```java
@Override
public boolean equsals(Object o) {
    throw new AssertionError(); // 호출 금지
}
```

* 싱글턴을 보장하는 클래스(인스턴스 통제 클래스, Enum)인 경우

## equals를 재정의 해야 하는 경우

객체 식별성(Object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 할때, \
equals를 재정의 해야한다. 또한 Object 메서드를 재정의 할 때는 반드시 아래 일반 규약을 따라야한다.

* **반사성**

null이 아닌 모든 참조 값 x에 대해 x.equals(x) 는 true 이다. 즉, 객체는 자기 자신과 비교했을 때 같아야 한다.

* **대칭성**

null이 아닌 모든 참조 값 A, B에 대해 A.equals(B) 가 참이면, B.euals(A)도 참이어야 한다.

```java
public final class **문자열비교**{
  private final String s;

  public **문자열비교**(String s) {
    this.s = Objects.requireNonNull(s);
  }

  @Override
  public boolean equals(Object o) {
    if(o instanceof **문자열비교**) {
      return s.equals((String) o)
    }

    **if(o instanceof String) { 
      return s.equalsIgnoreCase((String) o);
    }**
    return false;
  }
}
```

위 소스의 경우, 아래와 같은 대칭성 예외가 발생한다.

```java
문자열비교클래스 aaa = new 문자열비교클래스**("banana");
String bbb = "banana";

aaa.equals(bbb)  => true;
bbb.equsls(aaa)  => false;

aaa를 기준으로 bbb와 비교한 경우 참이나, bbb를 기준으로 aaa와 비교한 경우 거짓이다.**
```

대칭성 예외 해결을 위해서는 String과의 비교문은 삭제해야 한다.

* **추이성**

null이 아닌 참조값 A, B, C에 대해 A와 B가 같고, B와 C가 같으면, A와 C도 같아야 한다는 조건이다.

```java
class Point {
	int x;
	int y;
	
	@Override
  public boolean equals(Object o) {
		if(!(o instanceof Point)) return false;

		return x == (Point)o.x && 
						y == (Point)o.y;
	}
}

class ColorPoint extends Point {
  
  private final Color color;

  @Override
  public boolean equals(Object o) {
    if(!(o instanceof Point)) return false;

    //객체가 Point이면 x, y정보만 비교한다.
    if(!(o instanceof ColorPoint)) return o.equals(this);
    
    //객체가 ColorPoint이면 색상까지 비교한다.
    return super.equals(o) && this.color == ((ColorPoint) o).color;
  }
}
```

위 소스의 경우, 아래 같은 추이성 위반이 발생한다.

```java
ColorPoint a = new ColorPoint(1, 2, Color.RED);
Point b = new Point(1, 2);
ColorPoint c = new ColorPoint(1, 2, Color.BLUE);

******************a.equals(b) => true
b.equals(c) => true
a.equals(c) => false******************
```

* **일관성**

null이 아닌 참조값 A, B 에 대해 A.equals(B)를 반복해서 호출 해도 항상 같은 값이 나와야한다.

* **null-아님**

null이 아닌 모든 참조 값 A에 대해 A.equals(null)은 거짓이다.

## 그 외 equalse 재정의시 주의사항

* 리스코프 치한 원칙 위배 케이스

리스코프 치환 원칙; 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행할 수 있어야 한다.

```java
class Point {
	int x;
	int y;

	private static final Set<Point> 단위원 = Set.of(new Point(0, -1),
   new Point(0, 1),
   new Point(-1, 0),
   new Point(1, 0)
  );	

	public static boolean 반지름이1이냐?(Point p) {
    return 단위원.contains(p);
  }

  @Override
  public boolean equals(Object o) {
    if(o == null || o.getClass() != this.getClass()) {
      return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
  }
}

class ColorPoint extends Point {
  
  private final Color color;
}
```

위 소스의 경우 ColorPoint는 Point 클래스가 아니기 때문에 부모 클래스 Point의 “반지름이1이냐?” 메서드를 통과할 수 없다. 따라서 해당 equals 메서드는 리스코프 치한 원칙에 어긋나는 잘못된 재정의이다.

```java
ColorPoint cp = new ColorPoint(1, 0, Color.RED);
Point.onUnitCircle(cp) => false
```

## equals 메서드 재정의 단계별 정리

1. \== 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
   1. 해당 케이스에서 묵시적 null 체크가 가능하다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 검사한다.
   1. float와 double을 제외한 기본 타입 필드는 == 연산자로 비교
   2. 참조 타입 필드는 equals 메서드로 비교
   3. float와 double 필드는 Float.compare(), Double.compare로 비교한다.
5. null이 의심되는 필드는 Objects.equals(obj, obj)를 이용해 NullPointerException을 예방하자
