# 2차 피드백

@lxxjn0 requested changes on this pull request.

피드백 반영하시느라 수고하셨습니다!!

테스트는 너무 잘 짜주셨고 전체적은 코드 개선도 너무 잘 해주신 것 같아요!! 👍🏼 지난 번에 의견이 정확히 전달되지 않은 몇 가지 부분에 대한 추가적인 피드백 정도 남겨뒀는데 한번 확인하고 반영해주시면 좋을 것 같아요 :)

***

## 불용어

{% hint style="info" %}
불용어란? 없어도 의미 전달에 영향이 없는 단어

* a, an, the와 같은 접두어는 의미가 분면이 다를 때만 사용하도록 한다.
* 불용어를 사용하여 중복을 발생시키지 말자
  * product라는 클래스가 존재 할 때, ProductInfo, ProductData와 같은 개념이 구분되지 않는 클래스를 생성하여 중복을 발생시키지 않도록 한다.
{% endhint %}

{% embed url="https://kyeoneee.tistory.com/55" %}

In [src/main/java/domain/ListFactory.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181485229):

```
> @@ -0,0 +1,6 @@
+package domain;
+
+public interface ListFactory {
```

이름이 조금 모호한 것 같아요. 만약 '계산기에서 사용할 구분 단위의 생성 팩토리' 등의 의미라면 CalculatorTokenFactory 등의 이름도 좋을 것 같아요. 단순한 인터페이스의 이름으로는 어떠한 구현체들이 존재할 지 감을 잡기 어려울 것 같아요!!

***

## 코딩 컨벤션

In [src/main/java/domain/ListFactory.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181485557):

```
> @@ -0,0 +1,6 @@
+package domain;
+
+public interface ListFactory {
+
+    public <T> T extractToList(String[] splits);
```

T가 리스트 타입이 아니면 해당 메서드 명은 올바르지 않은 메서드 명 아닐까요? 이름을 지을 때는 불용어가 들어가지 않는 것이 좋아요. 구글이 자바 코딩 컨벤션을 한번 학습해보시는 걸 추천드릴게요!!

{% embed url="https://github.com/JunHoPark93/google-java-styleguide" %}

***

In [src/main/java/domain/NumberListFactory.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181485881):

```
> @@ -0,0 +1,17 @@
+package domain;
+
+import java.util.List;
+import java.util.stream.Collectors;
+import java.util.stream.IntStream;
+
+public class NumberListFactory implements ListFactory {
+
+    @Override
+    public List<Integer> extractToList(String[] splits) {
+        return IntStream.iterate(0, i -> i + 1)
+                .limit(splits.length)
+                .filter(i -> i % 2 == 0)
```

해당 조건문을 메서드로 추출하면 좀 더 의미를 명확히 드러낼 수 있을 것 같아요!!

***

In [src/main/java/domain/Operator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181486623):

```
> @@ -0,0 +1,36 @@
+package domain;
+
+import java.security.InvalidParameterException;
+import java.util.List;
+import java.util.function.BiFunction;
+
+public enum Operator {
+    PLUS("+", (left, right) -> left + right),
+    SUBTRACT("-", (left, right) -> left - right),
+    MULTIPLY("*", (left, right) -> left * right),
+    DIVIDE("/", (left, right) -> left / right);
+
+    private String operation;
+    private BiFunction<Integer, Integer, Integer> biFunction;
```

BinaryOperator 인터페이스를 활용하면 좀 더 적절한 인터페이스 사용이 될 수 있을 것 같아요!!

***

In [src/main/java/domain/Operator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181487605):

```
> +public enum Operator {
+    PLUS("+", (left, right) -> left + right),
+    SUBTRACT("-", (left, right) -> left - right),
+    MULTIPLY("*", (left, right) -> left * right),
+    DIVIDE("/", (left, right) -> left / right);
+
+    private String operation;
+    private BiFunction<Integer, Integer, Integer> biFunction;
+
+    Operator(String operation, BiFunction<Integer, Integer, Integer> biFunction) {
+        this.operation = operation;
+        this.biFunction = biFunction;
+    }
+
+    public Integer calculate(List<Integer> numbers) {
+        if (this.operation.equals("/") && numbers.get(1) == 0) {
```

`this == DIVIDE` 로 비교해도 될 것 같아요!! 추가로 해당 메서드에서 리스트로 넘기면 지금처럼 내부에서 0, 1 인덱스에 해당하는 값을 추출해서 연산을 하게 되는데 Operator 객체의 책임은 피연산자 2개를 연산하는 책임만 가지는 것이 적절하지 않을까요??

그런 맥락에서 파라미터는 해당 메서드의 역할에 맞게 피연산자 두 개만 받는게 좋을 것 같아요!! 지금은 '리스트로 넘겨지는 값의 0, 1 인덱스를 연산할 때 사용한다' 라는 내부적인 로직까지 알고 있어야만 해당 메서드를 사용할 수 있는데 `PLUS.calculate(0, 1)` 이라고 사용한다면 내부 로직을 알지 않더라도 두 인자는 연산을 하고자 하는 두 개의 값이라는 걸 알 수 있지 않을까요?

***

In [src/main/java/domain/Operator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181488133):

```
> +        for (Operator value : values()) {
+            if (value.operation.equals(operation)) {
+                return value;
+            }
+        }
+        throw new InvalidParameterException("유효한 연산자가 아닙니다.");
```

이런 로직의 경우

```
Arrays.stream(values())
    .filter(operator -> operator.symbol.equals(symbol)
    .findAny()
    .orElseThrow(() -> new InvalidParameterException());
```

과 같이 사용할 수도 있을 것 같아요!! 추가로 연산자의 기호와 관련된 부분이니 symbol이라는 이름도 좋지 않을까 생각합니다 :)

***

## 클래스 기능 분리

{% embed url="https://jx2lee.github.io/java-function-responsibility/" %}

In [src/main/java/domain/StringCalculator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181488482):

```
>              throw new IllegalArgumentException("유효한 표현식이 아닙니다.");
         }
     }
 
-    List<Integer> extractNumbers(String[] splits) {
-        List<Integer> stringNums = new ArrayList<>();
-
-        for (int i = 0; i < splits.length; i++) {
-            if (i % 2 != 0) {
-                continue;
-            }
-            stringNums.add(toInt(splits[i]));
-        }
-        return stringNums;
-    }
-
-    private Integer toInt(String stringNum) {
-        Integer num = null;
+    Numbers extractNumbers(String expression) {
```

아, 제가 말씀드린 부분은 StringCalculator에서 Numbers를 생성하는 메서드가 있을 필요가 없지 않냐? 였어요. main 메서드에서 Numbers를 생성하는 로직을 호출하면 되지 않을까요?

***

In [src/main/java/domain/StringCalculator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181484798):

```
> +    private Numbers numbers;
+    private Operations operations;
```

StringCalculator를 사용하는 main 메서드에서 해당 필드들을 로컬 변수로 가지고 있으면 괜찮지 않을까요?? 필드는 객체의 존재 의미를 나타내는 중요한 정보라고 생각해요. 계산기가 물론 연산자와 숫자를 가질 수 있지만 연산을 할 연산자와 숫자를 가지는 것은 어색한 것 같아요!

만약 계산기의 연산할 식이 달라질 때마다 계산기 객체를 매번 새롭게 생성해야 하는데 그건 좀 어색하지 않을까요? 덧셈 계산기, 사칙연산 계산기 등 계산기에서 계산 가능한 연산자를 필드로 가진다면 필드의 유무가 의미가 있게 사용될 것 같은데, 지금은 단순한 입력값에 대한 정보를 필드로 가지고 있는 것 같아 계산기라는 객체의 재사용 성이 떨어지는 것 같아요. 한번 이런 부분을 기준으로 객체의 필드를 고민해보시면 더 좋을 것 같아요 :)

***

In [src/main/java/domain/NumberListFactory.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1181486215):

```
> +    @Override
+    public List<Integer> extractToList(String[] splits) {
+        return IntStream.iterate(0, i -> i + 1)
+                .limit(splits.length)
+                .filter(i -> i % 2 == 0)
+                .mapToObj(i -> Integer.parseInt(splits[i]))
+                .collect(Collectors.toList());
+    }
+}
```

IntStream의 range 메서드가 있는 걸로 알고 있는데 해당 메서드의 인자로 0, splits.length를 넘겨주면 되지 않을까요?
