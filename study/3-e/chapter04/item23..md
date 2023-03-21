# item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.&#x20;

## 태그 달린 클래스

```java
package Chapter3.Item23;

public class Figure {
    
    enum Shape {RECTANGLE, CIRCLE}

    final Shape shape;

    double length;
    double width;

    double radius;

    // 원용 생성자
    public Figure(double radius) {
        this.radius = radius;
        this.shape = Shape.CIRCLE;
    }

    // 사각형용 생성자
    public Figure(double length, double width) {
        this.length = length;
        this.width = width;
        this.shape = Shape.RECTANGLE;
    }

    double area() {
        switch (shape) {
            case CIRCLE:
                return length * width;

            case RECTANGLE:
                return Math.PI * radius * radius;

            default:
                throw new AssertionError(shape);
        }
    }
}

```

## Figure 클래스를 계층구조 방식으로 구현

```java
package Chapter3.Item23;

public abstract class Figure02 {
    abstract double area();
}

class Circle extends Figure02 {

    final double radis;

    public Circle(double radis) {
        this.radis = radis;
    }

    @Override
    double area() {
        return Math.PI * radis * radis;
    }
}

class Rectangle extends Figure02 {

    final double length;
    final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}

```

## 핵심 정리

태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자.&#x20;

