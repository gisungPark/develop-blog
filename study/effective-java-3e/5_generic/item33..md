# item33. 타입 안전 이종 컨테이너를 고려하라

## 핵심 정리

컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. **하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.** 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)를 표현한 DatabaseRow 타입에서는 제네릭 타입인 Column\<T>를 키로 사용할 수 있다.



## 타입 안전 이종 컨테이너 패턴이란?

제네릭은 Set\<E>, Map\<K, V> 등의 컬렉션과 ThreadLocal\<T>, AtomicReference\<T> 등의 단일원소 컨테이너에도 흔히 쓰인다. 이런 모든 쓰임에서 매개변수화되는 대상은 원소가 아닌 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다.

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

해당 문제의 해법으로는 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다. 이렇게 하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해 줄 것이다. 이러한 설계 방식을 타입 안전 이종 컨테이너 패턴이라 한다.&#x20;

```java
public void Favorites {
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
컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 **타입 토큰**이라 한다.&#x20;
{% endhint %}













## 타입 안전 이종 컨테이너 패턴 구현



## 제약 사항





[^1]: 
