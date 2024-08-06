## public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### ✅ Point 클래스 예제
```java
class Point {
    public double x;
    public double y;
}
```
위의 클래스는 데이터 필드를 직접 접근할 수 있어, 캡슐화의 이점을 제공하지 못한다.

필드가 public으로 선언되어 있으면, 해당 클래스의 내부 표현을 변경할 수 없다. 예를 들어, x와 y 좌표를 다른 형식으로 저장하고 싶을 때, API를 변경하지 않고는 이를 구현할 수 없다.
그리고, 외부에서 필드 값을 직접 변경할 수 있어 클래스 내부의 불변식을 보장하기 어렵다.
마지막으로, 필드에 접근할 때 부수 작업(예: 값 검증, 로깅, 이벤트 발생 등)을 수행해야 할 경우, public 필드는 이러한 작업을 삽입할 수 있는 기회를 제공하지 않는다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

그래서 위와 같이 필드는 private으로 하고, public 접근자를 추가하여 데이터를 캡슐화할 수 있다.
public 클래스에서라면 이 방식이 맞다.

### ✅ public 클래스

패키지 바깥에서 접근할 수 있는 클래스라면, 필드를 private으로 하고 public 접근자를 제공하는 것이 좋다.
이렇게 하면 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.


### ✅ package-private 클래스, private 중첩 클래스
package-private 클래스 혹은 private 중첩 클래스는 필드를 노출해도 상관 없다.
그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.

필드를 노출해도 큰 문제가 되지 않는 이유는 다음과 같다.

1. 이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다.

접근자(getter)와 설정자(setter) 메서드를 사용하지 않음으로써, 코드가 더 간결해진다. 
필드에 직접 접근함으로써 불필요한 메서드 호출을 피하고, 코드의 가독성을 높일 수 있다.

2. 접근 범위 제한

- `패키지-프라이빗 클래스(package-private class)`는 동일 패키지 내에서만 접근할 수 있다.
따라서 필드를 노출하더라도 해당 패키지 내부에서만 사용된다. 
패키지 바깥의 코드에서는 이 클래스에 접근할 수 없으므로, 필드가 노출되어 있어도 외부에서 이를 직접 조작할 위험이 줄어든다.

- `프라이빗 중첩 클래스(private nested class)`는 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다.

### ⛔ Point, Dimension 클래스

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다.
대표적인 예가 `java.awt.package` 패키지의 Point와 Dimension클래스다.
내부를 노출한 Dimension 클래스의 심각한 성능 문제는 아직 해결되지 못했다.

```java
//point 클래스
public class Point {
    public int x;
    public int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

```

```java
//dimension 클래스
public class Dimension {
    public int width;
    public int height;

    public Dimension(int width, int height) {
        this.width = width;
        this.height = height;
    }
}

```

### ❓ 불변 필드를 노출한 public 클래스
만약 public 클래스의 필드가 불변이라면, 직접 노출할 때의 단점이 줄어들기는 하지만 좋은 생각은 아니다.

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException("Hour must be between 0 and 23");
        }
        if (minute < 0 || minute >= MINUTES_PER_HOUR) {
            throw new IllegalArgumentException("Minute must be between 0 and 59");
        }
        this.hour = hour;
        this.minute = minute;
    }
}

```

필드가 public final로 선언되어 있으면, 내부 표현을 변경할 수 없다.
예를 들어, hour와 minute을 다른 형식(예: long 타입의 시간 값 등)으로 저장하고 싶을 때, API를 변경하지 않고는 이를 구현할 수 없다.
이는 API를 사용하는 모든 클라이언트 코드를 수정해야 하는 부담을 초래한다.
그리고,
필드를 읽을 때 부수 작업(예: 값 검증, 로깅, 이벤트 발생 등)을 수행할 수 없다. 필드가 public으로 노출되어 있기 때문에, 필드에 접근할 때 메서드 호출을 통한 추가 작업을 삽입할 수 없다.

따라서 이러한 문제들로 여전히 캡슐화 원칙을 완전히 만족시키지는 못한다.

단 불변식은 보장할 수 있다. 예를들어, 다음과 같이 Time 클래스에서 hour와 minute 필드는 한 번 설정되면 변경되지 않으므로, 
각 인스턴스가 유효한 시간을 표현함을 보장할 수 있다.



### 📌 핵심 정리
- public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
- 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.


---

### 📌 Reference
- 이펙티브 자바


