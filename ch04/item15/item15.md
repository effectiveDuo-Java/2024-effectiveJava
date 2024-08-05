## 클래스와 멤버의 접근 권한을 최소화하라
잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다. 
오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다. 

이러한 **정보 은닉** 혹은 **캡슐화**라고 하는 개념은 소프트웨어 설계의 근간이 되는 원리다. 

### ✅ 정보 은닉의 장점
```java
//정보 은닉이 없는 경우
public class DataStore {
    public List<String> data = new ArrayList<>();
}

public class DataProcessor {
    public void processData(DataStore store) {
        // 직접 데이터 수정
        // 'DataStore'의 내부 구현이 변경되면 'DataProcessor'도 수정해야함. 
        store.data.add("new data");
    }
}
```
```java
//정보 은닉이 있는 경우
public class DataStore {
    private List<String> data = new ArrayList<>();

    public void addData(String newData) {
        data.add(newData);
    }

    public List<String> getData() {
        return new ArrayList<>(data);
    }
}

public class DataProcessor {
    public void processData(DataStore store) {
        // 내부 데이터 구조에 접근하지 않고, 공개된 인터페이스만 사용
        // DataStore의 내부 구현이 변경되더라도, DataProcessor는 수정할 필요가 없음. 따라서 두 클래스가 독립성을 띈다는 것을 의미함.
        store.addData("new data");
    }
}

```

- 시스템 개발 속도를 높인다(=독립적으로 개발되고 유지보수 가능하기 떄문). 
여러 컴포넌트를 병렬로 개발할 수 있기 때문이다.(= 여러 팀이 동시에 다양한 컴포넌트 개발 가능)
- 시스템 관리 비용을 낮춘다. 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적기 때문이다.
- 정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다.
완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 다음, 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있기 때문이다.**(=모듈화 및 독립성)**
- 소프트웨어 재사용성을 높인다.
- 큰 시스템을 제작하는 난이도를 낮춰준다. 시스템 전체가 아직 완성되지 않은 상태에서도 개별 컴포넌트의 동작을 검증할 수 있기 때문이다.

위 내용은 5개로 나눠져있지만, 모두 **모듈화 및 독립성**에 따른 장점을 다룬 내용이다.

자바는 정보 은닉을 위한 다양한 장치를 제공하는데, 그 중 `접근 제어 메커니즘`은 클래스, 인터페이스, 멤버의 접근성을 명시한다.
각 요소의 접근성은 그 요소가 선언된 위치와 `접근 제한자(private, protected, public)`로 정해진다.

##### 📌기본 원칙은 '모든 클래스와 멤버 접근성을 가능한 한 좁혀야 한다'는 것이다.

### ✅ 톱레벨 클래스, 인터페이스
톱레벨이란 가장 바깥에 있는 것. 

#### 접근 수준
- `public`: 공개 API로 이용할 수 있다.
- `package-private`: 해당 패키지 안에서만 사용할 수 있다. 

따라서, 패키지 외부에서 쓸 이유가 없다면 `package-private`으로 선언하자. 그러면 이들은 API가 아닌 내부 구현이 되어 언제든 수정 가능하다.

#### private static 중첩 클래스, 인터페이스
만약 package-private 톱레벨 클래스나 인터페이스를 한 클래스에서만 사용한다면, 
이를 사용하는 클래스 안에 private static 으로 중첩시켜보자.

```java
class OuterClass {
    private static class NestedClass {
        // 중첩 클래스 내용
        void display() {
            System.out.println("Nested class method");
        }
    }

    public void useNestedClass() {
        NestedClass nested = new NestedClass();
        nested.display();
    }
}

public class Test {
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.useNestedClass(); // 가능
        // OuterClass.NestedClass nested = new OuterClass.NestedClass(); // 불가능, 접근 불가
    }
}

```
이 코드에서 Test 클래스는 NestedClass에 직접 접근할 수 없으며, 오직 `OuterClass` 내부에서 `NestedClass`를 생성하고 사용할 수 있다.
이로 인해 `NestedClass`는 `OuterClass` 안에서만 사용 가능한 클래스가 된다.

이렇게 톱레벨로 두면 같은 패키지의 모든 클래스가 접근할 수 있지만, private static으로 중첩시키면 바깥 클래스 하나에서만 접근할 수 있다.

### ✅ 멤버 (필드, 메서드, 중첩 클래스, 중첩 인터페이스)

#### 📍 접근 수준
-`private` : 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.
-`package-private` : 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다. 
접근 제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준이다. (단, 인터페이스의 멤버는 기본적으로 public이 적용된다.)
-`protected` : package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다. (제약이 조금 따른다 [JLS, 6.6.2])
-`public` : 모든 곳에서 접근할 수 있다.

클래스의 공개 API를 세심히 설계한 후, 그 외의 모든 멤버는 private으로 만들자.
그런 다음 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여 private을 제거하여 `package-private`으로 풀어주자.

##### ❗️ 단, `Serializable`을 구현한 클래스에서는 private과 package-private 멤버 필드들이 의도치 않게 공개 API가 될 수 있음에 주의하자.
```java
import java.io.*;

class MyClass implements Serializable {
    private int secretNumber;
    String defaultAccessName;

    public MyClass(int secretNumber, String defaultAccessName) {
        this.secretNumber = secretNumber;
        this.defaultAccessName = defaultAccessName;
    }

    public int getSecretNumber() {
        return secretNumber;
    }

    public String getDefaultAccessName() {
        return defaultAccessName;
    }
}

public class SerializationDemo {
    public static void main(String[] args) {
        MyClass myObject = new MyClass(12345, "John Doe");

        try {
            // 객체 직렬화
            FileOutputStream fileOut = new FileOutputStream("object.ser");
            ObjectOutputStream out = new ObjectOutputStream(fileOut);
            out.writeObject(myObject);
            out.close();
            fileOut.close();

            // 객체 역직렬화
            FileInputStream fileIn = new FileInputStream("object.ser");
            ObjectInputStream in = new ObjectInputStream(fileIn);
            MyClass deserializedObject = (MyClass) in.readObject();
            in.close();
            fileIn.close();

            // 역직렬화된 객체의 필드 값 출력
            System.out.println("Secret Number: " + deserializedObject.getSecretNumber());
            System.out.println("Default Access Name: " + deserializedObject.getDefaultAccessName());

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```

1. 직렬화된 데이터 노출:
- 직렬화된 객체는 object.ser 파일에 저장되므로, 이 파일을 열어보면 객체의 모든 필드 값이 포함된 데이터를 볼 수 있다.
- 이는 MyClass의 `private` 필드 secretNumber와 `패키지 접근 수준` 필드 defaultAccessName이 외부에 노출되는 결과를 초래한다.

2. 역직렬화된 객체 접근:
- 역직렬화된 객체를 통해 `private` 필드에 접근할 수 있다.
- 위 코드에서는 `getSecretNumber()` 메서드를 통해 `private` 필드 `secretNumber`에 접근하고 있다.

따라서 직렬화와 역직렬화 과정에서 정보 은닉이 깨질 수 있다는 점 주의하자.


##### public 클래스에서는 멤버의 접근 수준을 package-private에서 protected로 바꾸는 순간 그 멤버에 접근할 수 있는 대상 범위가 엄청나게 넓어진다. 
##### public 클래스의 protected 멤버는 공개 API이므로 영원히 지원돼야 한다.

```java
package com.example.api;

public class ParentClass {
    //com.example.api 패키지 외부의 서브클래스에서도 접근 가능
    protected void doSomething() {
        ...
    }
}
```

```java
package com.example.client;

import com.example.api.ParentClass;

public class ChildClass extends ParentClass {
    @Override
    protected void doSomething() {
        ...
    }
}
```

이때 ParentClass의 `doSomething()` 메소드는 공개 API의 일부분이 되며, 
외부에서 이를 오버라이드하는 클래스가 있기 때문에 ParentClass의 `doSomething()` 메소드는 변경 없이 유지되어야 합니다.

`public` 클래스의 `protected` 멤버는 한번 공개되면 외부 코드의 안정성과 호환성을 위해 영원히 지원되어야 합니다.
따라서, `protected` 멤버의 수는 적을수록 좋다.

#### 📍 주의 사항

##### 1️⃣ 제약 사항: 상위 클래스 메서드 재정의

상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다.

이 제약은 **상위 클래스의 인스턴스**는 **하위 클래스의 인스턴스**로 대체해 사용할 수 있어야 한다는 규칙 `(리스코프 치환 원칙)`을 지키기 위해 필요하다. 
이 규칙을 어기면 컴파일 오류가 발생한다.

❌ 잘못된 코드 ❌ 

```java
//상위 클래스
public class Parent {
    public void display() {
        System.out.println("Parent display");
    }

//하위 클래스(잘못된 접근 수준)
public class Child extends Parent {
    // 컴파일 오류: display()의 접근 수준을 public보다 좁게 설정할 수 없음
    void display() {
        System.out.println("Child display");
    }
}
}

```

##### 2️⃣ 테스트 목적으로 접근 범위 확장 제한
단지 코드를 테스트하려는 목적으로 클래스 접근 범위를 넓히려 할 때가 있다. 
예를 들어 public 클래스의 private 멤버를 package-private까지 풀어주는 것은 허용할 수 있지만 그 이상은 안된다

```java
// Initial version with private member
public class Example {
    private int secretNumber;

    public Example(int secretNumber) {
        this.secretNumber = secretNumber;
    }

    private int getSecretNumber() {
        return secretNumber;
    }
}

// Testing within the same package
package com.example;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class ExampleTest {
    @Test
    public void testSecretNumber() {
        Example example = new Example(42);
        assertEquals(42, example.getSecretNumber());
    }
}

```

```java
public class Example {
    protected int secretNumber; // protected로 범위 확장

    public Example(int secretNumber) {
        this.secretNumber = secretNumber;
    }

    public int getSecretNumber() { // public
        return secretNumber;
    }
}

```
이렇게 변경하면, Example 클래스의 내부 상태가 불필요하게 외부로 노출되어, 클래스의 캡슐화가 깨지고 유지보수성이 떨어지게 된다.

이러한 이유로 클래스의 접근 범위를 테스트 목적으로 넓힐 때는 package-private까지 허용하고, 그 이상은 허용하지 않는 것이 좋다.

##### 3️⃣ public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.

필드가 가변 객체를 참조하거나 final이 아닌 인스턴스 필드를 public으로 선언하면 그 필드에 담을 수 있는 값을 제한할 힘을 잃게 된다.
가변 객체를 public 필드로 선언하면, 외부 코드에서 해당 객체의 상태를 직접 변경할 수 있기 때문이다.

또한, 필드가 수정될 때 다른 작업을 할 수 없게 되므로 **public 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다.**

**스레드 안전성 (Thread Safety)**
- 스레드 안전성: 여러 스레드가 동시에 객체에 접근해도 객체의 일관성이 유지되는 특성.
- 문제점: public 가변 필드를 갖는 클래스는 필드가 변경될 때 이를 제어하거나 동기화할 수 없습니다.

```java
public class MyClass {
    public int counter;
}

MyClass obj = new MyClass();

Runnable task = () -> {
    for (int i = 0; i < 1000; i++) {
        obj.counter++;
    }
};

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);
t1.start();
t2.start();

```
counter 필드에 대한 동시 접근이 제대로 제어되지 않아 예상치 못한 결과가 발생할 수 있다.


그리고 심지어 필드가 final이면서 **불변 객체를 참조**하더라도 문제가 있다. 
내부 구현을 바꾸고 싶어도 그 public 필드를 없애는 방식으로는 리팩터링할 수 없게 된다.

-> 불변 객체를 참조하는 final 필드를 public으로 선언하면, 필드 자체는 변경되지 않지만 클래스 설계를 변경하기 어렵다.

```java
public class MyClass {
    public final String name;

    public MyClass(String name) {
        this.name = name;
    }
}

```
위 예시에서 name 필드는 외부에 공개되어 있기 때문에, 내부 설계를 변경할 수 있는 여지가 줄어든다.
만약 name 필드를 private으로 변경하고, 이를 접근하기 위한 메서드를 추가한다면,

```java
public class MyClass {
    private final String name;

    public MyClass(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

```

기존의 public final String name 필드에 의존하던 코드가 모두 깨지게 됩니다. 
따라서 리팩토링이 어렵고, 기존 API를 유지하면서 내부 구현을 변경하기 어렵습니다.

##### 4️⃣ public 정적 필드 예외: 상수
해당 클래스가 표현하는 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 **상수**라면 `public static final` 필드로 공개해도 좋다.
단, 이런 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야 한다.

public static final 필드는 수정할 수 없으므로, 한 번 설정된 값이 변경되지 않는다.
이는 해당 상수가 불변성을 가지며, 프로그램 내에서 일관된 값을 제공함을 보장한다.

```java
public class MathConstants {
    // 클래스가 표현하는 추상 개념을 완성하는 상수
    public static final double PI = 3.14159;  // 원주율
    public static final double E = 2.71828;   // 자연로그의 밑

    // 불변 객체
    public static final BigDecimal BIG_PI = new BigDecimal("3.14159");
}
```

관례상 이런 상수의 이름은 대문자 알파벳으로 쓰이며 각 단어 사이에 밑줄 _ 을 넣는다.

##### 5️⃣ 배열 필드는 public static final로 공개하면 안 된다.
길이가 0이 아닌 배열은 모두 변경 가능하니 주의하자!
**클래스에서 public static final 배열 필드를 두거나, 이 필드를 반환하는 접근자 메서드를 제공해서는 안 된다.**
아래의 코드에서, 클라이언트가 배열의 내용을 수정할 수 있게 되므로 보안 허점이 존재한다

```java
// 잘못된 코드 - 보안 허점이 존재한다.
public static final Thing[] VALUES = {...};
```

2가지 해결방안이 있다.

1. private 배열로 만들고, public 불변 리스트를 추가한다.

```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES = 
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

```

위 방법은

Collections.unmodifiableList로 만든 리스트는 불변이므로, 외부에서 리스트의 내용을 수정할 수 없다.
그리고, 배열을 private로 선언함으로써, 클래스의 내부 구현이 외부에 노출되지 않는다.
마지막으로 public으로 제공되는 리스트는 읽기 전용이므로, 안전하게 외부에 데이터를 노출할 수 있다. 사용자가 의도치 않게 데이터를 수정하는 것을 방지한다.

는 장점이 있다.

2. private 배열로 만들고, 그 복사본을 반환하는 public 메서드를 추가한다. (방어적 복사)

```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

위 방법은

배열의 복사본을 반환함으로써, 원본 배열이 수정되는 것을 방지한다.
사용자는 복사본을 통해 데이터에 접근할 수 있지만, 원본 데이터는 안전하게 보호된다.
배열을 private로 선언하여, 클래스의 내부 구현이 외부에 노출되지 않도록 한다.

라는 장점이 있다.

### ✅ 자바 9: 모듈 시스템
모듈은 패키지들의 묶음이며, 공개(export)할 패키지들을 관례상 module-info.java 에 선언한다.
protected 혹은 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서는 접근할 수 없다.
이러한 모듈 시스템의 개념이 도입되면서 2가지 암묵적 접근 수준이 추가되었다.

숨겨진 패키지 안에 있는 public 클래스의

- `public` 멤버: public 수준과 같으나, 그 효과가 모듈 내부로 한정된다.
- `protected` 멤버: protected 수준과 같으나, 그 효과가 모듈 내부로 한정된다.

모듈 시스템을 활용하면 클래스를 외부에 공개하지 않으면서도 같은 모듈을 이루는 패키지 사이에서는 자유롭게 공유할 수 있다.

그러나, 모듈에 적용되는 새로운 두 접근 수준은 상당히 주의해서 사용해야 한다.

모듈의 JAR 파일을 자신의 모듈 경로가 아닌 애플리케이션의 클래스패스(classpath)에 두면,
그 모듈 안의 모든 패키지는 마치 모듈이 없는 것처럼 행동한다.
따라서 그 모듈 안의 모든 패키지는 (공개 여부와 상관없이) public 클래스의 모든 public 혹은 protected 멤버를 모듈 밖에서도 접근할 수 있게 된다.

아래 코드가 예시이다.
```java
// 모듈 A의 구조
module com.example.moduleA {
    exports com.example.moduleA.publicapi; // 공개 API 패키지
    // com.example.moduleA.internal 패키지는 공개하지 않음
}

package com.example.moduleA.publicapi;

public class PublicClass {
    public void publicMethod() {
        System.out.println("Public method in PublicClass");
    }
}

package com.example.moduleA.internal;

public class InternalClass {
    public void internalMethod() {
        System.out.println("Internal method in InternalClass");
    }
}

// 애플리케이션 클래스패스에서 접근 시도
package com.example.app;

import com.example.moduleA.internal.InternalClass;

public class MainApp {
    public static void main(String[] args) {
        InternalClass internalClass = new InternalClass();
        internalClass.internalMethod(); // 접근 가능 (잘못된 접근)
    }
}


```
이렇게 기존의 방식대로 클래스패스에 포함된 모든 클래스는 접근 제어 없이 사용할 수 있다. 따라서 잘못된 접근.

그러나 자바 9부터 모듈 시스템이 도입되면서, JDK 자체가 모듈화되어, 더욱 견고한 접근 제어가 가능해졌다.

**올바른 예시**
1. 모듈 정의: module-info.java 파일을 사용하여 모듈을 정의하고, 공개할 패키지를 명시한다.
2. 접근 제한: 모듈 경로를 사용하여 모듈 간의 접근을 제한.
```java
// module-info.java 파일 (com.example.moduleA 모듈)
module com.example.moduleA {
    exports com.example.moduleA.publicapi; // 공개 API 패키지
    // com.example.moduleA.internal 패키지는 공개하지 않음
}

//패키지 및 클래스
package com.example.moduleA.publicapi;

public class PublicClass {
    public void publicMethod() {
        System.out.println("Public method in PublicClass");
    }
}


package com.example.moduleA.internal;

public class InternalClass {
    public void internalMethod() {
        System.out.println("Internal method in InternalClass");
    }

//애플리케이션 모듈

module com.example.app {
    requires com.example.moduleA;
}

package com.example.app;

import com.example.moduleA.publicapi.PublicClass;

public class MainApp {
    public static void main(String[] args) {
        PublicClass publicClass = new PublicClass();
        publicClass.publicMethod(); // 접근 가능

        // InternalClass는 모듈 외부에서 접근 불가
        // com.example.moduleA.internal.InternalClass internalClass = new com.example.moduleA.internal.InternalClass(); // 컴파일 오류
    }
}


```

1. com.example.moduleA 모듈의 module-info.java 파일에서 com.example.moduleA.publicapi 패키지만 공개한다. com.example.moduleA.internal 패키지는 공개하지 않으므로, 외부에서 접근할 수 없다.

2. 모듈 의존성: com.example.app 모듈의 module-info.java 파일에서 com.example.moduleA 모듈을 requires 키워드로 명시한다.
이를 통해 com.example.app 모듈은 com.example.moduleA 모듈의 공개된 API에 접근할 수 있다.

3. 접근 제한: MainApp 클래스에서 PublicClass는 접근 가능하지만, InternalClass는 접근할 수 없다. 이는 모듈 시스템이 com.example.moduleA.internal 패키지를 공개하지 않았기 때문이다. InternalClass를 사용하려고 시도하면 컴파일 오류가 발생한다.

이렇게 자바 모듈 시스템을 활용하여 모듈 간의 명확한 접근 제어를 적용할 수 있다. 모듈 경로를 통해 모듈을 배치하고, module-info.java 파일을 통해 접근 가능한 패키지와 그렇지 않은 패키지를 명시적으로 정의함으로써, 모듈 간의 불필요한 접근을 방지하고, 코드의 안전성과 보안성을 높일 수 있다.


### 📌 핵심 정리
- 프로그램 요소의 접근성은 가능한 한 최소화하라. 꼭 필요한 것만 public API로 설계하자.
- 그 외에는 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야 한다.
- public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안 된다.


---

### 📌 Reference
- 이펙티브 자바
- [[Effective Java] 아이템14: comparable을 구현할지 고려하라](https://wisdom-cs.tistory.com/43)

