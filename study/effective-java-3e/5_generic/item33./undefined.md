# 슈퍼 타입 토큰

익명 클래스와 제네릭 클래스 상속을 사용한 타입 토큰으로, 제네릭이 소거되지 않는 특이한 케이스를 활용할 수 있다.\
그러나 이조차도 제네릭 타입이 아닌 경우 사용이 불가하다는 한계가 있다.&#x20;



아래 예제에서 String List와 Integer LIst를 구분지어 사용이 불가능한데, Neal Gafter가 고안한 슈퍼 타입 토큰으로 일부 해결이 가능하다.

```java
public static void main(String[] args) {
    Column column = new Column();
    
    column.put(List.class, Arrays.asList(1, 2, 3));
    column.put(List.class, Arrays.asList("a", "b", "c"));
     
    System.out.println(column.get(List.class));
}
>> [a, b, c]
```



## 슈퍼 타입 토큰

제네릭은 컴파일 단계에서 Object로 변경되고, 사용하는 쪽에서 caseting하는 코드가 삽입된다. 따라서 런타임 시점에 제네릭의 타입 정보를 확인할 수 없다.&#x20;

```java
public class GenericTypeInfer {

    static class Super<T> {
        T value;
    }

    public static void main(String[] args) throws NoSuchFieldException {
        Super<String> stringSuper = new Super<>();
        System.out.println(stringSuper.getClass().getDeclaredField("value").getType());
    }
}

>> class java.lang.Object
```

그러나 상속을 이용하면 달라진다. 하위 클래스의 인스턴스로부터 부모 타입의 타입 정보를 확인 할 수 있다.

```java
static class Super<T> {
    T value;
}

static class Sub extends Super<String> { << 클래스 상속 단계 추가

}

public static void main(String[] args) throws NoSuchFieldException {
    Sub sub = new Sub();
    Type type = sub.getClass().getGenericSuperclass();
    ParameterizedType pType = (ParameterizedType) type;
    System.out.println(pType.getActualTypeArguments()[0]); << 부모의 타입 파라미터 확인
}
>> class java.lang.String
```



익명 내부 클래스를 사용하면 위 코드의 하위 클래스를 선언하고 생성하는 번거로운 과정을 생략하고, 간결하게 부모의 제네릭 타입을 알아낼 수 있다.

```java
Type type = sub.getClass().getGenericSuperclass(); << 변경 전

Type type = (new Super<String>() {}).getClass().getGenericSuperclass(); << 변경 후
```



위 개념을 활용하면, 실체화 불가 타입인 List에도 타입 안전성을 가질 수 있다. 코드 전문은 아래와 같다.

```java
public class TypeRef <T> {
    private final Type type;

    protected TypeRef() {
        ParameterizedType superclass = (ParameterizedType) getClass().getGenericSuperclass();
        type = superclass.getActualTypeArguments()[0];
    }

    @Override
    public int hashCode() {
        return type.hashCode();
    }

    @Override
    public boolean equals(Object o) {
        return o instanceof TypeRef && ((TypeRef) o).type.equals(type);
    }

    public Type getType(){
        return type;
    }
}
```

```java
class Column {
    Map<TypeRef<?>, Object> map = new HashMap<>();

    public <T> void put(TypeRef<T> typeRef, T value){
        map.put(typeRef, value);
    }
    public <T> T get(TypeRef<T> typeRef) {
        return (T) map.get(typeRef);
    }

    public static void main(String[] args) {
        Column column = new Column();

        column.put(new TypeRef<List<String>>() {}, Arrays.asList("a", "b", "c"));
        column.put(new TypeRef<List<Integer>>() {}, Arrays.asList(1, 2, 3));


        column.get(new TypeRef<List<String>>() {}).forEach(System.out::println);
        column.get(new TypeRef<List<Integer>>() {}).forEach(System.out::println);

    }
}
>> a, b, c
>> 1, 2, 3
```



