# item45. 스트림은 주의해서 사용하라

## 스트림이란?

* 다양한 데이터 소스(컬렉션, 배열)를 표준화된 방법으로 다루기 위한 것
* 스트림의 핵심 추상 개념은 아래 두 가지가 있다.
  * 스트림
    * &#x20;데이터 원소의 유한 혹은 무한 시퀀스를 의미
    * 스트림의 원소는 컬렉션, 배열, 파일 정규표현식 등 어느 것이든 될 수 있다.
  * 스트림 파이프라인
    * 스트림 원소를 어떻게 처리할지 결정
    * 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 중간 연산

* 연산 결과가 스트림인 연산
* 예컨대 각 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소 걸러내는 연산 등이 있다.

### 종단 연산

* 마지막 중간 연산이 내놓은 스트림 가하는 최후의 연산을 말한다. (단 한번만 적용 가능)\
  원소를 정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력하는 등이 있다.&#x20;

```java
/* 중간 연산*/
1) sorted
Stream<Integer> sorted = operands.stream().sorted();

2) filter
Stream<Integer> integerStream = operands.stream().filter((value) -> value > 2);

3) map
list.map(s -> s.toUpperCase());


/* 종단 연산 */
1) forEach
intList.stream().forEach(System.out::println); // 1,2,3
intList.stream().forEach(x -> System.out.printf("%d : %d\n",x,x*x)); // 1,4,9

2) reduce
int sum = intList.stream().reduce((a,b) -> a+b).get();
System.out.println("sum: "+sum);  // 6

3) count(), min(), max() : 요소의 통계
intList.stream().count();
intList.stream().min(numberList);
intList.stream().max(numberList);
```

스트림 파이프라인은 지연 평가 된다. 연산은 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠이다. 종당 연산이 없는 스트림 파이프라인은 아무일도 하지 않는다.

> **지연 연산이란 간단히 말해 결과값이 필요할 때까지 계산을 늦추는 기법이다**. 즉, 눈앞에 코드가 주어졌을 때 곧바로 해당 코드를 실행하는 것이 아니라 실행 결과가 필요해지는 시점에 실행하도록 하는 것이다. 다만, 이러한 방식으로 코드가 동작하기 위해 내부적으로 준비해주는 작업이 필요하므로 무조건 효율적인 방식이라고는 보기 어렵다.&#x20;
>
> 이에 반해 어떠한 작업이 **즉시(eager)** 수행된다면 말 그대로 실행할 코드가 보이는 순간 곧바로 실행된다는 의미이다. 사실 특정 작업의 실행 결과가 향후 필요하다는 점이 확실하다면 미리 실행해놓는 것이 기본적으로 성능 측면에서 우월하다고 볼 수 있다.\
> \
> 출처: \[\[Java] 스트림: 지연 연산과 최적화]\([https://bugoverdose.github.io/development/stream-lazy-evaluation/](https://bugoverdose.github.io/development/stream-lazy-evaluation/))

## 스트림을 남용하지 말자

스트림 API는 다재다능하여 사실상 어떠한 계산이라도 해낼수 있다. **하지만 할 수 있다는 뜻이지, 해야 한다는 뜻은 아니다. 스트림을 잘못 사용하면 읽기 어렵고 유지보수가 힘들어진다.**&#x20;

### 아나그램 예제

아나그램이란 철자를 구성하는 알파벳이 같고 순서만 다른 단어를 말한다. 아래 예제는 사용자가 지정한 개수보다 원소 수가 많은 아나그램 그룹을 출력한다. 맵의 키는 그 단어를 구성하는 철자들을 알파벳순으로 정렬한 값이다.&#x20;

{% hint style="info" %}
예를 들어 "sample"의 키는 aelpst가 되고, "petals"의 키도 aelpst으로 이 두 단어는 아나그램이다.
{% endhint %}

#### 예제1. for문 활용 예제 (7\~17 라인)

{% code lineNumbers="true" %}
```java
public class Anagrams {
      public static void main(String[] args) throws IOException {
            File dictionary = new File(args[0]);
            int minGroupSize = Interger.parseInt(args[1]);
            
            Map<String, Set<String>> groups = new HashMap<>();
            try(Scanner s = new Scanner(dictionary)){
            	while(s.hasNext()){
                  String word = s.next();
                  groups.computeIfAbsent(alphabetize(word), 
                                    (unused) -> new TreeSet<>()).add(word);
                }
            }
            
            for(Set<String> group : groups.values()) {
            	if(group.size() >= minGroupSize)
              	System.out.println(group.size() + ":" + group);
             }
    }
    
    public static String alphabetize(String s){
    	char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
{% endcode %}

#### 예제2. stream 과용 예제

{% code lineNumbers="true" %}
```java
try(Strem<String> words = Files.lines(dictionary)) {
    words.collect( groupingBy(word -> word.chars().sorted()
                    .collect(StringBuilder::new, (sb, c) -> sb.append((char)c),
                    StringBuilder::append).toString()))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .map(group -> group.size() +": " + group)
                .forEach(System.out::println);    
                                
}
```
{% endcode %}

{% hint style="info" %}
groupingBy 메서드는 최대 3가지 파라미터를 받는 메서드들로 구성되어 있다.

1. classifier (Function\<? super T,? extends K> ): 분류 기준을 나타낸다.
2. mapFactory (Supplier) : 결과 Map 구성 방식을 변경할 수 있다.
3. downStream (Collector\<? super T,A,D>): 집계 방식을 변경할 수 있다.
{% endhint %}

예제 2번의 경우 확실히 짧지만 읽기는 어렵다. 특히 스트림에 익숙하지 않은 프로그래머라면 더욱 그럴 것이다. 이처럼 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.&#x20;

#### 예제3. 절충안

{% code lineNumbers="true" %}
```java
try(Strem<String> words = Files.lines(dictionary)) {
    words.collect( groupingBy(word -> alphabetize(word))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .forEach(g -> System.out.println(g.size() + " : " + g));    
}

 public static String alphabetize(String s){
    	char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
```
{% endcode %}

스트림을 전에 본 적 없더라도 이 코드는 이해하기 쉬울 것이다. 스트림 변수의 이름을 words로 지어 스트림 안의 각 원소가 word임을 명확히 했다. **이 스트림 파이프라인에는 중간 연산은 없으며, 종단 연산에서는 모든 단어를 수집해 맵으로 모은다.**&#x20;

이 맵은 단어들을 아나그램끼리 묶어놓은 것으로, 앞선 두 프로그램이 생성한 맵과 실직적으로 같다. 그 다음으로 이 맵의 values()가 반환한 값으로부터 새로운 Stream\<List\<String>> 스트림을 연다. 이 스트림의 원소는 물론 아나그램 리스트이다. 그 리스트 중 원소가 minGroupSize보다 적은 것은 필터링 돼 무시된다. 마지막으로, 종단 연산인 forEach는 살아남은 리스트를 출력한다.&#x20;

### char 타입을 지원하지 않는 stream

alphabetize 메서드도 스트림을 사용해 다르게 구현할 수 있으나, 기대했던것 보다 더 낮은 성능을 가질 가능성이 있다. 자바가 기본 타입인 char 용 스트림을 지원하지 않기 때문이다. 아래 예제를 참고하자.

```java
"Hello World!".chars().forEach(System.out::print);
// char가 아닌 정수값이 출력됨 : 739488237102..
```

올바른 print 메서드를 호출하려면 다음처럼 형변환을 명시적으로 해야한다. **하지만 char 값들을 처리할 때는 스트림을 삼가하는 편이 낫다.**&#x20;

```java
"Hello World!".chars().forEach(x -> System.out.print((char) x);
// char가 아닌 정수값이 출력됨 : 739488237102..
```

## 스트림은 항상 옳은가?

* 코드 가독성과 유지보수 측면에서 손해를 볼 수 있다.&#x20;
* 스트림내 내부 로직이 복잡해 디버깅이 어려워진다.

기존 코드를 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하자.

## 스트림을 언제 사용하는가?

### 스트림의 제약사항

스트림 파이프라인은 되풀이는 되는 계산을 주로 함수 객체(람다, 메서드 참조)를 이용한다. 이는 for문에서 사용되는 코드 블럭과는 차이점을 가진다.

* 스트림의 람다는 지역 변수를 수정할 수 없다.
  * 람다는 final이거나 사실상 final인 변수만 읽을 수 있다.
* return 문을 이용해 메서드에서 빠져나가거나, conrinue, break 등의 키워드로 반복을 건너뛰는게 불가하다.
* 또한, 메서드 선언에 명시된 검사 예외를 던질 수 ㅇ없다.

### 스트림 사용 권장

아래와 같은 로직이라면 스트림을 적용하기 좋은 후보이다.

* 원소들의 시퀀스를 일관되게 변환한다.
* 원소들의 시퀀스를 컬렉션에 모은다.
* 특정 조건을 만족하는 원소를 찾는다.

### 스트림 사용 불가

* 한 데이터가 파이프라인의 여러 단계를 통과하며, 이 데이터의 각 단계의 값들에 동시에 접근이 필요한 경우
  * 스트림 파이프라인은 스트림 원소를 다른 값에 매핑하고 나면 원래 값을 잃는 구조이기 때문이다.



### for문, 스트림 비교

스트림과 반복 중 어느 쪽을 써야 할지 바로 알기 어려운 작업도 많다. 다음은 카드 덱을 초기화하는 작업이다.

숫자와 무늬는 열거 타입이며, 이 두 집합의 원소들로 만들 수 있는 가능한 모든 조합을 계산하는 문제이다. 아래는 for-each 반복문을 중첩해서 구현한 코드이다.

{% code lineNumbers="true" %}
```java
List<Card> result = new ArrayList<Card>();
 
for(Suit suit : Suit.values()) {
    for(Rank rank : Rank.value) {
        result.add(new Card(suit, rank));
    }
}
```
{% endcode %}

다음은 스트림으로 구현한 코드이다.

```java
Stream.of(Suit.values())
        .flatMap(suit -> Stream.of(Rank.values())
                .map(rank -> new Card(suid, rank)))
        .collect(toList());
```

{% hint style="info" %}
flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다. 이를 평탄화(flattening)라고도 한다.
{% endhint %}

위 두 코드는 개인 취향과 프로그래밍 환경의 문제이다. 선호하는 방식을 사용하자.



## 핵심 정리

스트림, for문 사이 정답은 없으며 둘을 조합했을 때 가장 멋지게 해결된다. 어느 쪽이 나은지가 확연히 드러나는 경우가 많겠지만, 아니더라도 방법은 있다. 둘 다 해보고 더 나은 쪽을 선택해라.





