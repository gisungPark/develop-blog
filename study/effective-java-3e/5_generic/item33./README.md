# item33. 타입 안전 이종 컨테이너를 고려하라

제네릭은 Set\<E>, Map\<K, V> 등의 단일원소 컨테이너에 흔히 쓰인다. **이런 모든 쓰임에서 매개변수화되는 대상은 원소가 아닌 컨테이너 자신이다.** 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다.

그러나 이 장에서 기대하는 **이종 컨테이너는** 서로 다른 타입을 넣을 수 있는 컨테이너를 말한다.&#x20;

{% hint style="info" %}
컨테이너란? 다른 객체를 담을 수 있는 또 다른 객체를 말한다. List, Set, Queue, Map 등의 예가 있다.

[https://jeongsam.tistory.com/82](https://jeongsam.tistory.com/82)
{% endhint %}

<pre class="language-java" data-line-numbers><code class="lang-java">Map&#x3C;***, ***> map = new HashMap&#x3C;>();
// 기대 값 ===========
map.put(String, "사과");
map.put(String, "바나나");
map.put(Integer, 12);
map<a data-footnote-ref href="#user-content-fn-1">.</a>put(String, 72);   >> String 타입 키에 Integer 값을 넣으면 에러가 발생하길 기대

</code></pre>

## 타입 안전 이종 컨테이너 패턴

보통의 제네릭 컨테이너는 정의된 하나의 타입만 담을 수 있다. 하지만 종종 더 유연한 수단이 필요한 경우가 있다. 예컨대 데이터베이스의 행은 임의 개수의 컬럼을 가질 수 있는데, 해당하는 타입의 value만 넣을 수 있도록 만들 수 있는 컨테이너가 있으면 멋질 것이다.

특정한 타입, 예를 들어 문자열이면 문자열 값을 Map 컨테이너의 value로 넣고 싶고, 숫자면 숫자 값을 value로 넣는 컨테이너를 만들고 싶다고 하면 아래와 같은 구조를 쉽게 떠올릴 것이다.

```java
public class Database {
    List<Column> columns = new ArrayList<>();
}

class Column {
    // Map의 key를 class로 선언하여 특정한 타입 값을 받고, 
    // value는 다양한 타입 값을 받아와야하니 Object로 선언
    Map<Class, Object> map = new HashMap<>();

    public void put(Class classType, Object value){
        map.put(classType, value);
    }

    public Object get(Class classType) {
        return map.get(classType);
    }
}

Column column = new Column();
column.put(String, "사과");
column.put(String, "바나나");
column.put(Integer, 12);
column.put(String, 72);  << 에러 발생을 기대했으나, 정상 동작
```

그러나 위의 Column 객체는 기대와 다르게 타입 safe 하지 않을뿐더러 제네릭조차도 제대로 활용하지 못한 케이스이다.(map에서 커낼때, 형변환을 클라이언트가 직접 해야한다.)



### key 타입 매개변수화

해당 문제의 해법으로는 <mark style="color:orange;">**컨테이너 대신 Key에 타입을 선언함으로써 해결할 수 있다.**</mark> Key 값을 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다. **이렇게 하면 제네릭 타입 시스템이 값의 타입과 키가 같음을 보장해 줄 것이다**. 이러한 설계 방식을 <mark style="color:orange;">**타입 안전 이종 컨테이너 패턴**</mark>이라 한다.&#x20;

```java
class Column {
    Map<Class<?>, Object> map = new HashMap<>();

//  public void put(Class classType, Object value) {  << 수정 전
    public <T> void put(Class<T> classType, T value) { << 수정 후
        map.put(classType, value);
    }
//  public Object get(Class classType) {            << 수정 전
    public <T> T get(Class<T> classType) {          << 수정 후
        return classType.cast(map.get(classType));
    }
}

```

Map\<Class\<?>, Object> map 코드를 통해 Class 객체를 매개변수화한 key 값을 사용한다.

key가 매개 변수화되었다는 점만 빼면 일반 맵과 동일하다. 컬럼에 값을 담을 때, Class 객체를 알려주면 된다. 다음은 String, Integer 클래스를 저장, 검색 하는 예제이며, 메서드 단계에서 타입을 확인함으로써 타입 safe 해졌음을 확인 할 수 있다.

```java
public static void main(String[] args) {
     Column column = new Column();

     String s = column.put(String.class, "apple");
     Integer i = column.put(Integer.class, "1"); << 이제 컴파일 에러 발생, 타입 safe해졌다.
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

위의 Column 클래스는 완전한 타입 세이프 이종 컨테이너인가? \
그렇지 않다. 해당 클래스에는 알아두어야 할 두 가지 제약이 있다.&#x20;

**첫째, 악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로 타입으로 넘기면  Cloumn 인스턴스의 타입 안전성이 쉽게 깨진다.**

```java
 class Column {
    // Map의 key를 class로 선언하여 특정한 타입 값을 받고, 
    // value는 다양한 타입 값을 받아와야하니 Object로 선언
    Map<Class, Object> map = new HashMap<>();

    public void put(Class classType, Object value){
        map.put(classType, value);
    }

    public Object get(Class classType) {
        return map.get(classType);
    }
        
    //=======================
    //===== Main 함수 ========
    //=======================
    public static void main(String[] args) {
        Column column = new Column();
        column.put((Class)String.class, 120);  << 이슈 지점
    }
```

key값을 String 클래스의 로 타입인 Class로 형변환하여 넣으면, Integer 값인 120을 넣을 수 있게된다. 그렇다고 "정상적으로 동작하냐?" 하면 또 그건 아니다. <mark style="color:orange;">**런타임 시, get() 메서드에서 반환값을 형변환하는 과정에서 classCastException이 발생한다.**</mark>&#x20;

* 값을 넣고, 빼는 메서드 단계에 타입 체크 로직을 추가하자

아래 같이 수정하면, 값을 넣는 단계(put)에서 에러가 발생하도록 앞당길 수 있다. 그러나 이 역시도 런타임 에러로 컴파일 단계 에러를 확인할 수 없다. 클라이언트가 로 타입으로 넣기 시작하면 타입 안전성을 보장할 방법이 없다는 단점이 있다.

```java
//값을 넣고 빼기 전에 검사하는 로직을 추가하자.
public <T> void put(Class<T> clazz, T value){
//        map.put(clazz, value);           << 수정 전
        map.put(clazz, clazz.cast(value)); << 수정 후
}
public <T> T get(T clazz){
    return clazz.cast(map.get(clazz));
}    
```



**둘째, 실체화 불가 타입에는 사용 할 수 없다.**

```java
public static void main(String[] args) {
    Column column = new Column();
    
    column.put(List.class, Arrays.asList(1, 2, 3));
    column.put(List.class, Arrays.asList("a", "b", "c"));
     
    System.out.println(column.get(List.class));
}
>> [a, b, c]
```

* column.put(List.class, List.of("a", "b", "c");&#x20;
* column.put(List.class, List.of(1, 2, 3);

아래 두개를 동일하게 취급하는데 구분할 수는 없을까?

* column.put(List\<Integer>.class, List.of(1, 2, 3);
* column .put(List\<String>.class, List.of("a", "b", "c");&#x20;

List\<Integer>.class 리터럴은 없어, 구분할 수 없다. 슈퍼 타입 토큰을 이용하면 우회가능하나 이조차도 완벽한 우회로는 아니다.







## 핵심 정리

컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. **하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 **<mark style="color:orange;">**타**</mark><mark style="color:orange;background-color:yellow;">**입 안전 이종 컨테이너**</mark>**를 만들 수 있다.** 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)를 표현한 DatabaseRow 타입에서는 제네릭 타입인 Column\<T>를 키로 사용할 수 있다.





[^1]: 
