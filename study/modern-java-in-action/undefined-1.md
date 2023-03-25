# 람다 표현식

람다 표현식은 익명 클래스처럼 이름이 없는 함수면서 메서드를 인수로 전달할 수 있으므로 일단은 람다 표현식이 익명 클래스와 비슷하다고 하자.

이 장에서는 아래와 같은 것들을 학습한다.

* 람다 표현식을 어떻게 만드는지, 어떻게 사용하는지
* 자바 8 API 에 추가된 중요한 인터페이스와 형식 추론 기능
* 메서드 참조



## 람다란 무엇인가?

람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다.

```java
// 기존 코드
Comparator<Apple> byWeight = new Comparator<Apple> () {
    public int compare(Apple a1, Apple a2) {
        return a.getWeight().compareTo(a2.getWeight());
    }
}

// 람다를 이용한 코드
Comparator<Apple> byWeight 
            = (Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight());


```

### 람다의 구성

```java
(Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight());
```

위 코드에서 볼 수 있듯 람다는 세 부분으로 이루어진다.

* **파라미터 리스트**
  * Comparator의 compare 메서드 파라미터
* **화살표**
  * 화살표는 람다의 파라미터 리스트와 바디를 구분한다.
* **람다 바디**
  * 두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.

아래는 자바  8에서 지원하는 다섯 가지 람다 표현식 예제이다.

```java
(String s) -> s.length()        // 람다 표현식에는 return이 함축되어 있다.

(Apple a) -> a.getWeight() > 150

(int a, int b) -> {
    System.out.println("Result:");
    System.out.println(a + b);
}

() -> 42;

(Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight());
```



## 어디에, 어떻게 람다를 사용하는가?

함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다. 위 예제에서는 함수형 인터페이스 Predicate\<T> 를 filter메서드의 두 번째 인수로 람다 표현식을 전달했다.&#x20;

### 함수형 인터페이스&#x20;

<mark style="color:orange;">**하나의 추상 메서드를 지정하는 인터페이스를 함수형 인터페이스라 한다.**</mark>\ <mark style="color:orange;">****</mark>아래와 같은 예시가 있다.

\*많은 디폴트 메서드가 있더라도 추상 메서드가 하나면 오직 함수형 인터페이스이다.

```java
public interface Predicate<T> {
    boolean test(T t);
}

public interface Comparator<T> {
    int compare(T o1, T o2);
}

public interface Runnable {
    void run();
}

public interface ActionListener extends EventListener {
    void actionPerformed(ActionEvent e);
}
```

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급할** 수 있다.&#x20;

```java
Runnable r1 = () -> System.out.println("hello world1");

Runnable r2 = new Runnable() {
    public void run() {
        System.out.println("hello world2");
    }
}

public static void process(Runnable r) {
    r.run();
}


process(r1);
process(r2);
process(() -> System.out.println("hello world3");
```

### 함수 디스크립터&#x20;

* 메서드 시그니처; 메서드 이름과 매개변수 리스트의 조합을 말한다.&#x20;

새로운 자바 API를 살펴보면 함수형 인터페이스에 @FunctionalInterface 어노테이션이 추가 되었다. 해당 어노테이션은 함수형 인터페이스임을 가리키는 어노테이션이다. @FunctionalInterface로 인터페이스를 선언했지만 실제로 함수형 인터페이스가 아니면 컴파일 에러를 발생시키다.&#x20;

## 실행 어라운드 패턴

자원 처리(예를 들어 데이터베이스의 파일 처리)에 사용하는 순환 패턴은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다. 설정과 정리 과정은 대부분 비슷하다. 즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다. 아래와 같은 형식의 코드를 <mark style="color:orange;">**실행 어라운드 패턴**</mark>이라고 부른다.

```java
public String processFile() throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();
    }
}
```

<img src="../../.gitbook/assets/image (1).png" alt="" data-size="original">

\*중복되는 준비 코드와 정리 코드가 작업 A와 작업 B를 감싸고 있다.&#x20;

### 1단계: 동작 파라미터화를 기억하라.&#x20;

processFile의 동작을 파라미터화 해보자. BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 ProcessFile 메서드로 동작을 전달해야한다.&#x20;

```java
public String processFile(Process p) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(b);
    }
}

String result = processFile((BufferReader br) -> br.readLine() + br.readLine());
```

### 2단계: 함수형 인터페이스를 이용해서 동작 전달

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

public String processFile(BufferedReaderProcessor p) throws IOException {
}
```

### 3단계: 동작 실행

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}
```

### 4단계: 람다 전달

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());

String towLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 함수형 인터페이스, 형식 추론

자바 8 라이브러리 설계자들은 java.util.function 패키지로 여러가지 새로운 함수형 인터페이스를 제공한다. 이 절에서는 Proedicate, Consumer, Functino 인터페이스를 설명하며, 더 다양한 함수형 인터페이스를 소개한다.

### Predicate

```java
public <T> List<T> filter(List<T> list, Predicate<T> p) {
    ArrayList<T> result = new ArrayList<>();
    for(T t : list) {
        if(p.test(t)) {
            result.add(t);
        }
    }
    return result;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

### Consumer

java.util.function.Consumer\<T> 인터페이스는 제네릭 형식 T를 인수로 받아서 void를 반환하는  추상메서드 accept 를 정의한다.&#x20;

```java
public <T> void foreach(List<T> list, Consumer<T> c) {
    for(T t : list) {
        c.accept(t);
    }
}
foreach(Arrays.asList(1,2,3,4,5), (Integer i) -> System.out.println(i));
```

### Function

java.util.function.Function\<T, R> 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다.&#x20;

```java
public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    ArrayList<R> result = new ArrayList<>();
    for(T t : list) {
        result.add(f.apply(t));
    }

    return result;
}

 List<Integer> ret = functionTest01.map(Arrays.asList("lambdas", "in", "action"),
                                              (String s) -> s.length());
```

### 기본형 특화

위 3개의 함수형 인터페이스 외에도 특화된 형식의 함수형 인터페이스도 있다.&#x20;

자바의 모든 형식은 참조형 아니면 기본형에 해당한다. 하지만 제네릭 파라미터에는 참조형만 사용할 수 있다. 함수형 인터페이스 내에서 기본형을 참조형으로 변환하는 오토 박싱은 과정은 많은 비용이 소모되는 과정이다. 박식한 값은 기본형을 감싸는 래퍼며 힙에 저장된다. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요하다.&#x20;

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.&#x20;

```java
IntPredicate evenNumber = (int i) -> i % 2 == 0;
System.out.println(evenNumber.test(1000));


Predicate<Integer> oddNumbers = (Integer i) -> i % 2 == 0;
System.out.println(oddNumbers.test(1000));
```

일반적으로 특정 형식을 입력으로 받는 함수형 인터페이스의 이름 앞에는 DoublePredicate, IntConsumer, LongBinaryOperator 처럼 형식명이 붙는다.

### 형식 검사, 형식 추론, 제약

#### 형식 검사

앞서 람다로 함수형 인터페이스의 인스턴스를 만들 수 있다고 언급했다. 람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않다. 따라서 람다 표현식을 더 제대로 이해하려면 람다의 실제 형식을 파악해야한다.

#### 형식 추론

자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)을 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다. 즉 대상형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다. 결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다. 즉 자바 컴파일러는 다음처럼 람다 파라미터 형식을 추론할 수 있다.&#x20;

```java
// 파라미터 a에는 형식을 명시적으로 지정하지 않았다. 
List<Apple> greenApples = filter(inventory, apple -> GREEN.equals(apple.getColor()));
```

여러 파라미터를 포함하는 람다 표현식에서는 코드 가독성 향상이 더 두드러진다. 예를 들어 다음은 Comparator 객체를 만드는 코드다.

```java
// 형식을 추론하지 않음
Comparator<Apple> c = 
                (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
                
// 형식을 추론함
Comparator<Apple> c = 
                (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
                
```

상황에 따라 명시적으로 형식을 포함하는 것이 좋을 때도 있고 형식을 배제하는 것이 가독성을 향상시킬 때도 있다. 개발자 스스로 어떤 코드가 가독성을 향상시킬 수 있는지 결정해야 한다.&#x20;

#### 지역 변수 사용

람다 표현식에서는 익명 함수처럼 <mark style="color:orange;">**자유 변수**</mark>(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이와 같은 동작을 <mark style="color:orange;">**람다 캡처링**</mark>이라고 부른다.&#x20;

```java
int portNumber = 122;
Runnable r = () -> System.out.println(portNumber);
```

하지만 자유 변수에도 약간의 제약이 있다. 지역 변수는 명시적으로 final로 선언되어 있거나 final로 선언된 변수와 똑같이 사용어야 한다. 즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다.

예를 들어 다음 예제는 portNumber에 값을 두 번 할당하므로 컴파일 할 수 없다.

```java
int portNumber = 122;
Runnable r = () -> System.out.println(portNumber);
portNumber = 33;

java: local variables referenced from a lambda expression must be final or effectively final
```

#### 지역 변수의 제약

인스턴스 변수와 지역 변수는 태생부터 다르다. 인스턴스 변수는 힙에 저장되는 반면 지역 변수는 스택에 위치한다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. 따라서 자바 구현에서는 원래 변수를 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다.&#x20;

## 메서드 참조

메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

inventory.sort(comparing(Apple:getWeight());
```

메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이라고 생각할 수 있다. 예를 들어 람다가 '이 메서드를 직접 호출해'라고 명령한다면 메서드를 어떻게 호출해야 하는지 설명을 참조하기보다는 메서드명을 직접 참조하는 것이 편리하다. 이 때 명시적으로 메서드명ㅇ르 참조함으로써 가독성을 높일 수 있다.&#x20;

메소드 참조는 세 가지 유형으로 구분할 수 있다.

*   정적 메서드 참조

    예를 들어 Integer의 parseInt 메서드는 Integer::parseInt로 표현할 수 있다.
*   다양한 형식의 인스턴스 메서드 참조

    예를 들어 String의 length 메서드는 String::length로 표현할 수 있다.
*   기존 객체의 인스턴스 메서드 참조

    Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고, Transaction 객체에는 getValue 메서드가 있다면, 이를 expensiveTransaction::getValue라고 표현할 수 있다.&#x20;





## 람다 만들기&#x20;

