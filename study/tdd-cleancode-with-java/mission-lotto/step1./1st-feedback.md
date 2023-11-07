# 1차 피드백

@lxxjn0 requested changes on this pull request.

기성님, 안녕하세요 :)

이번 미션 구현하시느라 수고하셨습니다!! 그래도 Enum도 적절히 활용하시고 코드가 전반적으로 잘 분리가 된 느낌이에요!! 제가 몇 가지 부분에 대해서 피드백 남겨두었는데 한 번 확인하시고 피드백 반영 부탁드릴게요 :)

***

In [README.md](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179874043):

```
> +## 기능 요구 사항
+- [x] 문자열로부터 공백으로 구분된 숫자와 연산자를 분류한다.    
+- [x] 덧셈 연산을 계산한다.  
+- [x] 뺄셈 연산을 계산한다.  
+- [x] 곱셈 연산을 계산한다.  
+- [x] 나눗셈 연산을 계신한다. (단, 정수로 떨어지는 경우로 한정한다.)   
+- [x] 둘 이상의 연산자가 존재할 경우, 앞에서부터 차례로 계산한다.  
+- [x] 숫자, 연산자에 대한 예외 처리를 한다.  
+- [x] 사용자로부터 문자열을 입력받는다.  
```

구현할 기능 목록을 상세히 작성해주셨네요 👍🏼

***

In [src/main/java/domain/Calculator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179877070):

```
> @@ -0,0 +1,21 @@
+package domain;
+
+public class Calculator {
```

연산자의 연산 수식을 나타내니 Operator라는 이름도 좋을 것 같아요!! 그리고 추가로 자바에서 Enum을 잘 활용하면 더 좋은 코드로 개선할 수 있을 것 같아요!! 👍🏼

[https://techblog.woowahan.com/2527/](https://techblog.woowahan.com/2527/)

***

In [src/main/java/domain/Calculator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179877599):

```
> +package domain;
+
+public class Calculator {
+    public static int plus(int firstNum, int secondNum) {
+        return firstNum + secondNum;
+    }
+
+    public static int subtract(int firstNum, int secondNum) {
+        return firstNum - secondNum;
+    }
+
+    public static int multiply(int firstNum, int secondNum) {
+        return firstNum * secondNum;
+    }
+
+    public static int divide(int firstNum, int secondNum) {
```

secondNum이 0과 같은 값이 들어오면 해당 로직이 문제가 발생할 것 같아요!!

***

In [src/main/java/domain/Numbers.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179877764):

```
> +    private static final int DEFAULT_RETURN_NUMBER_COUNT = 2;
+    private Deque<Integer> numbers;
```

상수와 변수 사이에 공백을 주면 가독성을 개선할 수 있어요!!

***

In [src/main/java/view/InputView.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179889762):

```
> @@ -0,0 +1,8 @@
+package view;
+
+public class InputView {
```

KeyboardInput 클래스를 나눈 것이 잘못된 건 전혀 아닌데 현재는 각각의 클래스의 책임이나 역할이 적어서 같이 한 클래스로 써도 괜찮을 것 같아요!! ㅎㅎ 그리고 혹시 View는 객체로 사용하시고 KeyboardInput은 클래스로 사용하셨는데 이렇게 비슷한 두 클래스의 사용 방식을 다르게 가져간 이유가 있을까요?

***

In [src/main/java/domain/Operation.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179890024):

```
> @@ -0,0 +1,28 @@
+package domain;
+
+import java.security.InvalidParameterException;
+import java.util.function.BiFunction;
+
+public enum Operation {
```

앗, 여기 이미 Enum으로 관리하고 있었네요!! 그렇다면 각각의 operator가 하는 행위를 함께 관리해보는 것은 어떨까요?

***

In [src/main/java/domain/Operation.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179890122):

```
> +        for (Operation value : values()) {
+            if (value.operation.equals(operation)) {
+                return value;
+            }
+        }
+        throw new InvalidParameterException("유효한 연산자가 아닙니다.");
```

stream을 활용하면 더 깔끔하게 해결할 수 있을 것 같아요!!

***

In [src/main/java/domain/StringCalculator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179890567):

```
> +    private Numbers numbers;
+    private Operations operations;
```

해당 값을 꼭 필드로 가지고 있을 필요는 없을 것 같은데 어떻게 생각하시나요? 밖에서 Numbers와 Operations를 만들어서 calculate 메서드에 전달해준다면 해당 필드들은 따로 가지고 있을 필요가 없을 것 같아요!! :)

***

In [src/main/java/domain/StringCalculator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179891013):

```
> +
+import java.util.ArrayList;
+import java.util.List;
+
+public class StringCalculator {
+
+    private Numbers numbers;
+    private Operations operations;
+
+    public void readExpression(String exp) throws IllegalArgumentException {
+        validExpression(exp);
+        numbers = new Numbers(extractNumbers(exp.split(" ")));
+        operations = new Operations(extractOperation(exp.split(" ")));
+    }
+
+    private void validExpression(String exp) {
```

메서드나 변수 이름에 축약어를 지양하시면 좋을 것 같아요!! 특정한 축약어에 대한 이해가 없는 사람이 본다면 코드의 가독성이 떨어져 보일 것 같아요!!

***

In [src/main/java/domain/StringCalculator.java](https://github.com/next-step/java-lotto/pull/2998#discussion\_r1179892249):

```
> +    private Numbers numbers;
+    private Operations operations;
+
+    public void readExpression(String exp) throws IllegalArgumentException {
+        validExpression(exp);
+        numbers = new Numbers(extractNumbers(exp.split(" ")));
+        operations = new Operations(extractOperation(exp.split(" ")));
+    }
+
+    private void validExpression(String exp) {
+        if (exp == null || exp.isBlank()) {
+            throw new IllegalArgumentException("유효한 표현식이 아닙니다.");
+        }
+    }
+
+    List<Integer> extractNumbers(String[] splits) {
```

Factory와 같은 클래스를 만들어 주시는 건 어떨까요? 지금은 계산기에서 너무 많은 역할을 담당하고 있는 것 같아요!!

