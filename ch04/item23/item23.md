## 태그 달린 클래스보다는 클래스 계층구조를 활용하라
### 태그 달린 클래스
태그 달린 클래스란 **여러 타입의 인스턴스를 하나의 클래스에서 처리하기 위해 특정 필드나 변수를 사용하여 타입을 구분하는 방식**을 말합니다.  
여기서 **태그**란 클래스의 인스턴스가 어떤 형태나 타입을 가지고 있는지 나타내기 위해 사용되는 특별한 필드를 의미합니다.  

```java
public class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 도형의 타입을 나타냄
    final Shape shape;

    // 다음 필드는 사각형일 때만 사용
    double length;
    double width;

    // 다음 필드는 원일 때만 사용
    double radius;

    // 사각형을 위한 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    // 원을 위한 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    public double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```  

여러 구현이 한 클래스에 혼합돼 있어서 발생하는 **문제점**들이 많습니다.  
- 유효하지 않은 상태 허용: 태그 달린 클래스는 클래스의 인스턴스가 항상 유효한 상태가 아닐 수 있습니다.
    - `예시`: `RECTANGLE` 타입 인스턴스에서 `radius` 필드는 필요가 없습니다.  
- 가독성과 유지보수성 저하: 코드가 복잡해지고, 새로운 형태를 추가하거나 기존 형태를 변경할 때 코드의 많은 부분을 수정해야합니다. 즉, 오류 발생 가능성이 높습니다.  
    - `예시`: 예를 들어, 정사각형 모양을 추가한다고 했을 때, switch 문을 변경해야합니다.

### 클래스 계층구조  
태그 달린 클래스들이 가지던 문제점들을 해결하기 위해 클래스 계층구조가 등장했습니다.  
이는 다형성을 활용하여 각 타입에 대한 구체적인 클래스를 별도로 정의하고, 공통된 인터페이스나 추상 클래스 및 추상 메서드를 정의하는 방식입니다.  

기존 태그 달린 클래스로 구현한 코드를 클래스 계층구조로 변경한 코드입니다.  
```java
abstract class Figure {
    abstract double area();
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}
```

클래스 계층 구조는 태그 달린 클래스들의 단점들을 날려버립니다.  
- 유효한 상태 유지: 각 클래스는 자기 자신에게 의미 있는 필드만을 가지기에 항상 유효한 상태를 유지합니다.  
    - `예시`: `Rectangle` 클래스의 `radius`, `enum Shape` 필드 등이 이에 해당합니다.
- 가독성과 유지보수성 향상: 새로운 형태의 도형을 추가할 때, 새로운 클래스를 정의하면 되므로 코드 수정 범위가 줄어듭니다.  

---

### 📌 Reference
- 이펙티브 자바
