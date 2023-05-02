# item39. 명명 패턴보다 애너테이션을 사용하라

## 명명 패턴

명명 패턴이란 변수나 함수 이름을 일관된 방식으로 작성하는 패턴을 말한다. 이러한 명명 패턴은 전통적으로 도구나 프레임워크에서 특별히 다뤄야 할 프로그램 요소의 구분을 위해 사용되어 왔다. (ex. Junit2에서는 테스트 메서드 이름을 test 로 시작하게끔 함)

### 명명 패턴의 단점

* 오타가 나면 안된다.
* 명명 패턴을 의도한 곳에서만 사용할 거라는 보장이 없다.
* 명명 패턴을 적용한 요소를 매개변수로 전달할 마땅한 방법이 없다.

## 명명 패턴의 대안, 애너테이션

애너테이션은 위 명명패턴의 단점을 모두 해결해주는 멋진 개념은로,  JUnit도 버전 4부터 전면 도입했다.



아래 예제들은 애너테이션의 동작 방식을 보여주고자 직접 제작한 작은 테스트 프레임워크를 사용할 것이다.&#x20;

{% code title="마커 애너테이션 타입 선언" %}
```java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}

```
{% endcode %}

이처럼 애너테이션 선언에 다른 애너테이션을 메타에너테이션이라 한다.

`@Retention(RetentionPolicy.RUNTIME)` 메타애너테이션은 @Test가 런타임에도 유지되어야 한다는 표시이다. 만양 이 메타애너테이션을 생갹하면 테스트 도구는 @Test를 인식할 수 없다.

`@Target(ElementType.METHOD)` 메타애너테이션은 @Test가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다. 따라서, 클래스 선언, 필드 선언 등 다른 프로그램 요소에는 달 수 없다.&#x20;

### 마커 애너테이션

{% code title="마커 애너테이션을 사용한 프로그램 예" lineNumbers="true" %}
```java
public class Sample {
    @Test public static void m1() { }
    
    public static void m2() { }
    
    @Test
    public static void m3() {
        throw new RuntimeException("실패");
    }
    public static void m4() { }
    
    @Test public void m5() { }
    
    public static void m6() { }
    
    @Test
    public static void m7() {
        throw new RuntimeException("실패");
    }
    
    public static void m8() { }
}

```
{% endcode %}

위 코드는 마커 애너테이션을 사용한 프로그램 예제이다. 총 4개의 테스트 메서드 중 1개는 성공, 2개는 실패, 1개는 잘못 사용했다. 그리고 @Test를 붙이지 않은 나머지 4개의 메서드는 테스트 도구가 무시할 것이다.&#x20;

@Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다 그저 이 애너테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다.&#x20;

{% code lineNumbers="true" %}
```java
public class RunTests {

    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;

        Class<?> testClass = Class.forName("chapter6.item39.Sample");

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable excCause = wrappedExc.getCause();
                    System.out.println(m + "실패 : " + excCause);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공 : %d, 실패: %d%n", passed, tests - passed);
    }
}

실행 결과 ===============
public static void chapter6.item39.Sample.m3()실패 : java.lang.RuntimeException: 실패
잘못 사용한 @Test: public void chapter6.item39.Sample.m5()
public static void chapter6.item39.Sample.m7()실패 : java.lang.RuntimeException: 실패
성공 : 1, 실패: 3


```
{% endcode %}

### 특정 예외 체크 애노테이션

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

위 코드는 Class 리터럴을 애너테이션의 매개변수 값으로 사용하는 예제이다. 애너테이션의 매개변수의 타입은 Class\<? extends Throwable> 이다. 여기서 이 와이드카드 타입은 Trowable을 확장한 클래의 Class 객체라는 뜻으로, 모든 예외 타입을 다 수용한다.&#x20;

아래에서 이 애너테이션을 다룰 수 있는 테스트 도구를 보여 준다. main 7\~13 라인 이 변경 되었다.&#x20;

{% code lineNumbers="true" %}
```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        passed++;
    } catch (InvocationTargetException wrappedExc) {
    
    /* 변경점 start ============== */
        Throwable excCause = wrappedExc.getCause();
        Class<? extends Throwable> excType = 
                    m.getAnnotation(ExceptionTest.class).value();
        if (excType.isInstance(excCause)) {
            passed++;
            continue;
        }
        System.out.printf("테스트 %s 실패 : 기대한 예외 %s, 발생한 예외 : %s\n", 
                    m, excType.getName(), excCause);
    /* 변경점 end ============== */
    
    } catch (Exception exc) {
        System.out.println("잘못 사용한 @Test: " + m);
    }
}
```
{% endcode %}

`@Test` 애너테이션용 코드와 비슷하나, 한 가지 차이점을 보인다. 그서은 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는 데 사용한다는 것이다.

### 하나 이상의 예외 체크 애노테이션

앞선 예제에서 한 걸음 더 들어가, 예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다. `@ExceptionTest` 애너테이션의 매개 변수 타입을 Class 배열로 수정해 보자.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MultiExceptionTest {
    Class<? extends Throwable>[] value();
}
```

배열 매개변수를 받는 애너테이션용 문법은 아주 유연하다. 원소가 여럿이 배열을 지정할 때는 다음과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해 주기만 하면 된다.

{% code lineNumbers="true" %}
```java
@MultiExceptionTest({ArithmeticException.class, 
            NullPointerException.class})
public static void m1() {
    List<String> list = new ArrayList<>();
    list.addAll(5, null);
}

```
{% endcode %}

다음은 `@MultiExceptionTest`를 지원하도록 테스트 러너를 수정한 모습이다.

{% code lineNumbers="true" %}
```java
catch (InvocationTargetException wrappedExc) {
    Throwable excCause = wrappedExc.getCause();
    Class<? extends Throwable>[] excTypes 
                    = m.getAnnotation(MultiExceptionTest.class).value();
    for (Class<? extends Throwable> excType :
            excTypes) {
        if (excType.isInstance(excCause)) {
            passed++;
            break;
        }
    }
}
```
{% endcode %}

### @Repeatable 메타에너테이션

자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다. 배열 매개변수를 사용하는 대신 애너테이션에 @Repeatable 애너테이션을 다는 방식이다.&#x20;

단, 아래와 같은 주의점이 있다.

* `@Repeatable`에 이 컨테이너 애너테이션의 Class 객체를 매개변수로 전달해야 한다.
* 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야한다.
* 컨테이너 애너테이션 타입에는 적절한 보전 정책(`@Retention`)과 적용 대상(`@Target)`을 명시해야 한다.

{% code lineNumbers="true" %}
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)        << @Repeatable 선언
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface ExceptionTestContainer {
    ExceptionTest[] value();
}

```
{% endcode %}

이제 배열 방식 대신 반복 가능 애너테이션 적용 예제를 살펴보자.

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doubleBad() {
    ...
}
```

{% code lineNumbers="true" %}
```java
for (Method m : testClass.getDeclaredMethods()) {
    if (m.isAnnotationPresent(ExceptionTest.class) 
                || m.isAnnotationPresent(ExceptionTestContainer.class)) {
        tests++;
        try {
            m.invoke(null);
            passed++;
        } catch (InvocationTargetException wrappedExc) {
            Throwable excCause = wrappedExc.getCause();
            int oldPassed = passed;
            ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
            for (ExceptionTest excTest : excTests) {
                if (excTest.value().isInstance(excCause)) {
                    passed++;
                    break;
                }
            }
        } catch (Exception exc) {
            System.out.println("잘못 사용한 @Test: " + m);
        }
    }
}
```
{% endcode %}

반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러 번 달때의 코드 가독성을 높여보았다. 이 방식으로 코드의 가독성을 개선할 수 있다면 이 방식을 사용하도록 하자. 하지만 애너테이션을 선언하고 이를 처리하는 부분에서는 코드 양이 늘어나며, 특히 처리 코드가 복잡해져 오류가 날 가능성이 커짐을 명심하자.

## 핵심 요약

이번 아이템의 테스트 프레임워크는 아주 간단하지만 애너테이션이 명명 패턴보다 낫다는 점을 확실히 보연준다. 테스트는 애너테이션으로 할수 있는 일중 극히 일부일 뿐이다.&#x20;

도구 제작자를 제외하고는, ㅇ애너테이션 타입을 직접 정의할 일은 겅의 없다. 하지만 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다. **IDE나 정적 분석 도구가 제공하는 애너테이션을 사용하면 해당 도구가 제공하는 진단 정보의 품질을 높여줄 것이다.**&#x20;
