# item29. 이왕이면 제네릭 타입으로 만들라

## 제네릭 클래스로 변환하기.

아래 스택 클래스를 제네릭 타입으로 만들어보자. 일반 클래스를 제네릭 클래스로 만드는 첫 단계는 클래스 선언에 타입 매개변수를 추가하는 일이다.&#x20;

```java
public class Stack {
    
    private Object[] elements;
    private int size = 0;
    
    private static final int DEFAULT_INITIAL_CAPACITY = 15;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object o) {
        elements[size++] = o;
    }
    
    public Object pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
        ...   
}
```

추가한 소스는 아래와 같다.

```java
public class Stack <E>{

    private E[] elemnts;
    
    public void push(E e) {
        ...
    }    
    public E pop() {
        ...
    }
}
```

이 단계에서 대체로 하나 이상의 오류가 뜨는데, 여기서는 아래와 같다.

아이템 28에서 설명한 것처럼, E와 같은 신체화 불가 타입으로는 배열을 만들 수 없다. 배열을 사용하는 코드를 제네릭으로 만들려 할 때는 이 문제가 항상 발복을 잡을 것이다. 적절한 해결책은 두 가지다.

* 첫째, Object 배열을 생성한 다음 제네릭 배열로 형변환 한다.
* 둘째, elements 필드의 타입을 E\[] 에서 Object\[]로 변경한다.

```
generic array creation
elements = new E[DEFAULT_INITIAL_CAPACITY];
           ^     
```



### 첫째, Object 배열을 생성한 다음 제네릭 배열로 형변환 한다.

이제 컴파일러는 오류 대신 경고를 내보낼 것이다.

컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없지만 우리는 할 수 있다. 따라서 이 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을 우리 스스로 확인해야 한다.&#x20;

* 문제의 배열 elements는 private 필드에 저장되고, 클라이언트로 변환되거나 다른 메서드에 전달되는 일이 전혀 없다.&#x20;
* push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다.

때문에 이 비검사 형변환은 확실히 안전하다. 비검사 형변환이 안전함을 직접 증명했다면 범위를 최소로 좁혀 @SuppressWarnings 애너테이션으로 해당 경고를 숨기자.(아이템27) 여기서는 생성자가 비검사 배열 생성 말고는 하는 일이 없으니 생성자 전체에서 경고를 숨겨도 좋다.&#x20;

```java
@SuppressWarnings("unchecked")
public Stack() {
        elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY];
}
```

### 둘째, elements 필드의 타입을 E\[] 에서 Object\[]로 변경한다.

제네릭 배열 생성 오류를 해결하는 두 번째 방법은 elements 필드의 타입을 E\[]에서 Obejct\[]로 변경하는 것이다. 이렇게 하면 첫 번째와는 다른 오류가 발생한다.&#x20;

```
found: Object, required: E
    E result = elements[--size];
                        ^
```

배열이 반환한 원소를 E로 형변환하도록 한다. E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. 이번에도 마찬가지로 우리가 직접 증명하고 경고를 숨길 수 있다. pop 메서드 전체에서 경고를 숨기지 말고, 아이템 27의 조언을 따라 비검사 형변환을 수행하는 할당문에서만 숨겨보자.

```java
public E pop() {

    @SuppressWarnings("unchecked") E result = (E) elements[--size];
```







## 핵심정리

클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 제네릭 타입을 이용하여 형변환 없이도 사용할 수 있도록 하라. 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.
