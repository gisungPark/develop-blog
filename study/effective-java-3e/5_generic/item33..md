# item33. 타입 안전 이종 컨테이너를 고려하라

## 타입 안전 이종 컨테이너 패턴

제네릭은 Set\<E>, Map\<K, V> 등의 단일원소 컨테이너에 흔히 쓰인다. **이런 모든 쓰임에서 매개변수화되는 대상은 원소가 아닌 컨테이너 자신이다.** 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다.

그러나 이 장에서 기대하는 **이종 컨테이너** 서로 다른 타입들을 넣을 수 있는 컨테이너를 말한다.&#x20;

{% hint style="info" %}
java.util 라이브러리에는 컨테이너(container) 클래스 들이 있으며, 그것의 기본 타입들은 List, Set, Queue, Map 이다.

[https://jeongsam.tistory.com/82](https://jeongsam.tistory.com/82)
{% endhint %}

<pre class="language-java" data-line-numbers><code class="lang-java">// 기대 값 ===========
map.put(String, "사과");
map.put(String, "바나나");
map.put(Integer, 12);
map<a data-footnote-ref href="#user-content-fn-1">.</a>put(String, 72);   // String 타입 키에 Integer 값을 넣으면 에러가 발생하길 기대


// 실제 ===========
Map&#x3C;Class, Object> map = new HashMap&#x3C;>();
map.put(String, "사과");
map.put(String, "바나나");
map.put(Integer, 12);
map.put(String, 72);
>> 기대 값과 다르게 모두 정상 동작

</code></pre>

하지만 더 유연한 수단이 필요한 경우가 있다. 예컨테 데이터베이스의 행은 임의 개수의 열을 가질 수 있는데, 모두 열을 타입 안전하게 이용할 수 있다면 멋질 것이다.&#x20;

해당 문제의 해법으로는 <mark style="color:orange;">**컨테이너 대신 키를 매개변수화한 다음,**</mark> <mark style="color:orange;">**컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다.**</mark> 이렇게 하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해 줄 것이다. 이러한 설계 방식을 **타입 안전 이종 컨테이너 패턴**이라 한다.&#x20;

```java
public void Favorites {  // Favorites이라는 명칭의 컨테이너
    private Map<Class<?>, Object> map = new HashMap<>();
    
   public <T> void put(Class<T> clazz, T value){}
   public <T> T get(T clazz){}
   
}    
```

Map\<Class\<?>, Object> map 코드를 통해 Class 객체를 매개변수화한 키로 사용한다. 키가 매개 변수화되었다는 점만 빼면 일반 맵과 동일하다. 클라이언트는 즐겨 찾기를 저장하거나 얻어올 때 Class 객체를 알려주면 된다. 다음은 String, Integer class를 저장, 검색, 출력하고 있다.

```java
 private Map<Class<?>, Object> map = new HashMap<>();

    public <T> void put(Class<T> clazz, T value){
        map.put(clazz, value);
    }
    public <T> T get(T clazz){
        return (T)map.get(clazz);
    }

    public static void main(String[] args) {
        Favorites fv = new Favorites();
        fv.put(String.class, "aaaa");
        fv.put(Integer.class, 72);

        System.out.println(fv.get(String.class));
        System.out.println(fv.get(Integer.class));

    }

```



{% hint style="info" %}
* 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 **타입 토큰**이라 한다.&#x20;
* 특정 타입의 클래스 정보를 넘겨서 타입 안전성을 꿰하도록 코드를 작성하는 기법을 **TypeToken**이라고 한다.

[https://yangbongsoo.gitbook.io/study/super\_type\_token](https://yangbongsoo.gitbook.io/study/super\_type\_token)
{% endhint %}

{% hint style="info" %}
리터럴은 데이터 그 자체를 뜻 한다. 변수에 넣는 변하지 않는 데이터를 의미하는 것이다.

`int a = 1;`

`int` 앞에 `a`는 변수이고, 여기서의 `1`은 리터럴이다.

[https://yoo11052.tistory.com/50](https://yoo11052.tistory.com/50)
{% endhint %}

## 제약 사항

그럼 위의 Favorites 클래스는 완전한 타입 세이프 이종 컨테이너인가? \
해당 클래스에 알아두어야 할 두 가지 제약이 있다.&#x20;

**첫째, 악의적인 클라이언트가 Class 객체를 (제네릭이 아닌) 로 타입으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨진다.**

```java
  public void Favorites {  // Favorites이라는 명칭의 컨테이너
    private Map<Class<?>, Object> map = new HashMap<>();

    public <T> void put(Class<T> clazz, T value){
        map.put(clazz, value);
    }
    public <T> T get(T clazz){
        return (T)map.get(clazz);
    }    
        
    //=======================
    //Main 함수 =======================
    public static void main(String[] args) {
        Favorites fv = new Favorites();
        fv.put((Class)String.class, 120);
    }
    
    

```

key값을 String 클래스의 로 타입인 Class 형변환을하여 넣으면, Integer 값인 120을 넣을 수 있게된다. 그렇다고 "정상적으로 동작하냐?" 하면 <mark style="color:orange;">**또 그건 아니다 get() 메서드에서 반환값을 형변환하는 과정에서 classCastException이 발생한다.**</mark>&#x20;

```java
//값을 넣고 빼기 전에 검사하는 로직을 추가하자.
public <T> void put(Class<T> clazz, T value){
        map.put(clazz, clazz.cast(value));
}
public <T> T get(T clazz){
    return clazz.cast(map.get(clazz));
}    
```

위와 같이 수정하면, 넣는 단계(put)에서 에러가 발생하도록 앞당길 수 있다. 그러나 이 역시도 런타임 에러로 컴파일 단계에서 잡을 수 없다. 클라이언트가 로 타입으로 넣기 시작하면 타입 안전성을 보장할 방법이 없다.



**둘째, 실체화 불가 타입에는 사용 할 수 없다.**

```java
Favorites fv = new Favorites();

fv.put(List.class, List.of("a", "b", "c");
List list = fv.get(List.class)
list.foreach(System.out::println);
>>a
>>b
>>c

```

```java
Favorites fv = new Favorites();

fv.put(List.class, List.of("a", "b", "c");
fv.put(List.class, List.of(1, 2, 3);
List list = fv.get(List.class)
list.foreach(System.out::println);

>>1
>>2
>>3
```

* fv.put(List.class, List.of("a", "b", "c");&#x20;
* fv.put(List.class, List.of(1, 2, 3);

아래 두개를 동일하게 취급하는데 구분할 수는 없을까?

* fv.put(List\<Integer>.class, List.of(1, 2, 3);
* fv.put(List\<String>.class, List.of("a", "b", "c");&#x20;

List\<Integer>.class 리터럴은 없어, 구분할 수 없다. 슈퍼 타입 토큰을 이용하면 우회가능하나 이조차도 완벽한 우회로는 아니다.







## 핵심 정리

컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. **하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 **<mark style="color:orange;">**타**</mark><mark style="color:orange;background-color:yellow;">**입 안전 이종 컨테이너**</mark>**를 만들 수 있다.** 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)를 표현한 DatabaseRow 타입에서는 제네릭 타입인 Column\<T>를 키로 사용할 수 있다.





[^1]: 
