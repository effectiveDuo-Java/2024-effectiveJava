## 변경 가능성을 최소화하라
불변 클래스란 그 인스턴스의 내부 값을 수정할 수 없는 클래스이다.
불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.

### ✅ 자바 플랫폼 라이브러리의 불변 클래스
1. String
2. 기본 타입의 박싱된 클래스들
    ex) Integer, Long, Double
3. BigInteger
4. BigDecimal

이러한 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.

### ✅ 불변 클래스 만들기 위한 규칙(5가지)

1. 객체의 상태를 변경하는 메서드를 제공하지 않는다.
2. 클래스를 확장할 수 없도록 한다. 상속을 막는 대표적인 예로는 클래스를 `final`로 선언한다.
3. 모든 필드를 `final`로 선언한다. 
4. 모든 필드를 `private`로 선언한다.
    필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다. 보통 기본 타입이나 불변 객체를 참조하는 필드를 public final로만 선언해도 불변 객체가 되지만, 이렇게 하면 다음 릴리스에서 내부 표현을 바꾸지 못하므로 권하지 않는다.
    이렇게 public으로 필드를 직접 공개하는 경우, 해당 필드의 자료형이나 구현 방식이 변경되면, 이를 사용하는 모든 클라이언트 코드도 함께 수정해야 하기 떄문이다.
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행하라.

    **방어적 복사란?**
    객체의 내부 가변 상태를 외부로 노출할 때, 원본 객체를 직접 반환하지 않고, 그 복사본을 반환하는 기법이다.

    다음은 생성자, 접근자에서의 방어적 복사 예이다.
    ```java
    public class Person {
    private final Date birthDate;

    public Person(Date birthDate) {
        // 방어적 복사
        this.birthDate = new Date(birthDate.getTime());
    }

    //접근자(getter)
    public Date getBirthDate() {
        // 방어적 복사
        return new Date(birthDate.getTime());
    }
    }

    ```
    생성자에서 Date 객체를 받아서 내부 필드에 저장할 때, 전달된 Date 객체의 복사본을 만들어 저장한다.
    이렇게 하면 외부에서 전달된 Date 객체를 수정하더라도 Person 객체의 내부 상태는 변경되지 않는다.

    그리고, getBirthDate 메서드는 birthDate의 복사본을 반환한다.
    이렇게 하면 반환된 Date 객체를 외부에서 수정하더라도 Person 객체의 birthDate 필드에는 영향을 미치지 않는다.

### ✅ 불변 클래스 예시: 복소수 클래스

```java
// 불변 복소수 클래스
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}


```

📌 **final 클래스**
    - 확장이 불가능하도록 Complex 클래스를 final로 선언하였다.

📌 **re, im 필드**
    - 모든 필드는 final, private으로 선언한다.

📌 **사칙연산 메서드 (plus, minus, times, dividedBy)**

    - 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다.
    - 객체의 값을 변경한 것이 아니라 새로운 객체를 반환한다는 사실을 강조하기 위해, 메서드 이름으로 동사(add) 대신 전치사(plus)로 사용했다.
    - 함수형 프로그래밍: 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴. 이 방식으로 프로그래밍하면 코드에서
        불변이 되는 영역의 비율이 높아지는 장점이 있다.

### ✅ 불변 객체 장점

#### - 근본적으로 스레드 안전(thread-safe)하여 동기화할 필요가 없다.
다른 스레드에 영향을 끼치지 않으므로 안심하고 공유할 수 있다.
방어적 복사도 필요 없다. 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미가 없다.
따라서, clone 메서드나 복사 생성자를 제공하지 않는 게 좋다. 

#### - 불변 객체끼리는 내부 데이터를 공유할 수 있다.

```java
import java.util.Arrays;

public final class MyBigInteger {
    private final int sign;  // 부호: 1 (양수), -1 (음수)
    private final int[] magnitude; // 크기: 배열로 숫자를 저장

    // 생성자
    public MyBigInteger(int sign, int[] magnitude) {
        this.sign = sign;
        this.magnitude = magnitude;
    }

    // negate 메서드: 부호를 반대로 하고, 크기는 그대로 유지
    public MyBigInteger negate() {
        return new MyBigInteger(-sign, magnitude);
    }

    @Override
    public String toString() {
        return (sign == -1 ? "-" : "") + Arrays.toString(magnitude);
    }

    public static void main(String[] args) {
        // 예시 숫자: 1234 (양수)
        int[] magnitude = {1, 2, 3, 4};
        MyBigInteger positive = new MyBigInteger(1, magnitude);
        MyBigInteger negative = positive.negate();

        System.out.println("Original: " + positive);
        System.out.println("Negated: " + negative);

        // magnitude 배열이 동일한지를 확인
        System.out.println("Are magnitude arrays the same? " + 
                           (positive.magnitude == negative.magnitude));
    }
}


```
ex)
Original: [1, 2, 3, 4]
Negated: -[1, 2, 3, 4]

이 예시는 negate 메서드를 사용하여 새로운 MyBigInteger 객체를 생성할 때, 숫자의 크기를 나타내는 magnitude 배열이 원본 객체와 공유됨을 보여준다.
즉, 부호는 변경되었지만, 숫자의 크기를 나타내는 배열은 **동일한 메모리 위치를 참조**하고 있다.

#### - 객체를 만들 때 불변 객체들을 구성요소로 사용하면, 불변식을 유지하기 훨씬 수월하다.
좋은 예시로, **맵의 키**와 **집합(set)**의 원소로 불변객체를 사용하는 방법이 있다. 
맵이나 집합은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정은 하지 않아도된다.

#### - 불변 객체는 그 자체로 실패 원자성(failure atomicity)를 제공한다.
💡 실패 원자성: 메서드에서 예외가 발생한 후에도 그 객체는 여전히 유효한 상태여야 한다는 성질.
-> 불변 객체의 메서드는 내부 상태를 바꾸지 않으니 이 성질을 만족한다.


### ✅ 불변 객체 단점
- 값이 다르면 반드시 독립된 객체로 만들어야 한다.
- 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다.

예를 들어,
백만 비트짜리 biginteger에서 비트 하나를 바꿔야 한다고 해보자.

```java
BigInteger moby=...;
moby = moby.flipBit(0);

```
flipBit 메서드는 새로운 BigInteger 인스턴스를 생성한다. 원본과 단지 한 비트만 다른 백만 비트짜리 인스턴스를 말이다. 
이 연산은 BigInteger의 크기에 비례해 시간과 공간을 잡아먹는다.
그래서 `BitSet`을 활용한다. 이 아이도 BigInteger 처럼 임의의 길이의 비트 순열을 표현하지만, **가변**이다.
BigSet 클래스는 원하는 비트 하나만 상수 시간 안에 바꿔주는 메서드를 제공한다.

```java
BitSet moby = ...;
moby.flip(0);

```

#### 📌 해결 방법 (2가지)
1. 다단계 연산(multistep operation)을 정확히 예측할 수 있다면, package-private 가변 동반 클래스(companion class)를 둔다.

자바 플랫폼 라이브러리에서 이에 해당하는 대표적인 예가 `String` 클래스이다.
String 불변 클래스 안에서 String의 가변 동반 클래스인 `StringBuilder`를 활용한 예시를 들어보자.

```java

//String만 사용할 경우
public class StringExample {
    public static void main(String[] args) {
        String result = "";
        //다단계 문자열 수행
        result += "Hello, ";
        result += "World!";
        result += " How are you today?";

        System.out.println(result); 
    }
}

```
String은 불변 객체이기 때문에 += 연산을 할 때마다 새로운 String 객체가 생성된다.
이렇게 하면 중간 단계에서 불필요한 객체가 많이 생성될 수 있다.

```java

//StringBuilder 활용
public class StringBuilderExample {
    public static void main(String[] args) {
        StringBuilder builder = new StringBuilder();
        //append를 통해 기존 문자열에 새로운 문자열을 효율적으로 추가할 수 있다. 이 과정에서 새로운 객체를 생성하지 않고, 내부 배열을 수정한다.
        builder.append("Hello, ");
        builder.append("World!");
        builder.append(" How are you today?");

        String result = builder.toString();  // 최종 결과를 불변 String으로 변환
        System.out.println(result);  // Output: Hello, World! How are you today?
    }
}

```

이렇게 `StringBuilder`를 사용하면 다단계 문자열 조작을 수행하는 동안 불필요한 String 객체 생성을 피할 수 있다.
마지막에 toString() 메서드를 호출하여 최종 결과를 **String 객체로 변환**한다. 
이렇게 하면 불변성을 유지하면서도 성능을 최적화할 수 있다.

2. 이렇게 다단계 연산을 예측할 수 없다면, 클래스를 public으로 제공하는 게 최선이다.

### ✅ 클래스 상속 방지하는 방법
클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야함을 기억하자.
가장 쉬운 방법은 `final` 클래스로 선언하는 법이지만, 더 유연한 방법이 있다.

그것은 바로
#### 모든 생성자를 private 혹은 package-private으로 만들고, public 정적 팩터리를 제공한다.

```java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    ...
}

```

패키지 바깥의 클라이언트에서 바라본 이 불변 객체는 사실상 final이다.
public이나 protected 생성자가 없으니 다른 패키지에서는 이 클래스를 확장하는 게 불가능하기 때문이다.

그리고 이런 정적 팩토리 방식은 다수의 구현 클래스를 활용한 유연성을 제공한다.



### 📌 핵심 정리
- getter가 있다고 해서 무조건 setter를 만들지는 말자.
데이터를 읽는 메서드(`getter`)는 필요에 따라 제공하더라도, 데이터를 수정하는 메서드(`setter`)는 고심해보자.
- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
- 단순한 값 객체는 항상 불변으로 만들자.
    무거운 값 객체도 불변으로 만들 수 있는지 고심하여야 한다.
- 성능 때문에 불변으로 만들 수 없다면, 불변 클래스와 쌍을 이루는 `가변 동반 클래스`를 `public` 클래스로 제공하도록 하자.
- 불변으로 만들 수 없는 클래스는 변경할 수 있는 부분을 최소한으로 줄이자.
    다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
    확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안 된다.

---

### 📌 Reference
- 이펙티브 자바


