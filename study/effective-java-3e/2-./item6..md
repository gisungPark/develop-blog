# item6. 불필요한 객체 생성을 피하라

## **의도치 않게 인스턴스를 생성하는 케이스**

```java
/** 예제01 **/
String s = new String("bikini"); // 절대 사용하지 말 것!

String s = "bikini"; // 새로운 String 인스턴스를 만드는 대신, 동일한 문자열 리터럴을 사용하는
									   // 객체가 존재한다면 재사용한다. 
```

## 정적 팩터리 메서드를 활용하지 않는 패턴

```java
/** 예제02 **/
Boolean.valueOf(String)

public static Boolean valueOf(boolean b) {
   return (b ? TRUE : FALSE);
}
```

## “비싼 객체”를 재사용없이 반복 생성하는 경우

메서드 내부에서 만들어진 정규표현식용 Pattern 인스턴스는, 한번 사용 후 버려져 곧 가비지 컬렉션 대상이 된다.

```java
/** 개선 전 **/
static boolean isRomanNumeral(String s) {
	return s.mathces("^(?=.)M*(C[MD]|D?C{0,3})");
}

/** 개선 후 **/
public class RomanNumerals {
	private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})");

	static boolean isRomanNumeral(String s) {
		return ROMAN.matcher(s).matches();
	}

}
```

## 의도치 않은 오토 박싱을 통한 객체 생성

```java
private static long sum() {
	Long sum = 0L;
	for ( long i = 0; i <= Integer.MAX_VALUE; i++) {
		sum += i;
	}
	return sum;
}
```

아이템06은 “객체 생성은 비싸니 피해야 한다”를 말하는게 아니다. 재사용하면 안되는 객체를 재사용함으로써 발생하는 피해가 객체를 반복 생성했을 때의 피해보다 훨씬 크다. 방어적 복사에 실패하면 언제 터져 나올지 모르는 버그와 보안 구멍으로 이어지지만, 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 준다.

### 💡**어댑터 패턴을 이용한 객체 생성시 주의**

객체가 불변이 아니라면, 재사용에 따른 위험이 따른다.

책에서 든 예를 Map과 keySet을 살펴보면 keySet 은 동일 객체를 반환하고, 이는 다른 곳에 영향을 주고 받을 수 있다. 1

```java
Map<Integer, String> map = new HashMap<>();
    map.put(1, "hello");
    map.put(2, "my name is");
    map.put(3, "9");

    Set<Integer> keySet1 = map.keySet();

    System.out.println("map 출력 :" + map);
    System.out.println("keySet1 출력 :" + keySet1);

    map.remove(3);
    Set<Integer> keySet2 = map.keySet();
    System.out.println("map 출력 :" + map);
    System.out.println("keySet1 출력 :" + keySet1);
    System.out.println("keySet2 출력 :" + keySet2);

    keySet1.remove(2);
    Set<Integer> keySet3 = map.keySet();
    System.out.println("map 출력 :" + map);
    System.out.println("keySet1 출력 :" + keySet1);
    System.out.println("keySet2 출력 :" + keySet2);
    System.out.println("keySet3 출력 :" + keySet3);
    
    /*
    map 출력 :{1=hello, 2=my name is, 3=9}
    keySet1 출력 :[1, 2, 3]
    map 출력 :{1=hello, 2=my name is}
    keySet1 출력 :[1, 2]
    keySet2 출력 :[1, 2]
    map 출력 :{1=hello}
    keySet1 출력 :[1]
    keySet2 출력 :[1]
    keySet3 출력 :[1]
    */
```

[Java String 에 대해 깊게 파고들어 보자\~!](https://aljjabaegi.tistory.com/465)
