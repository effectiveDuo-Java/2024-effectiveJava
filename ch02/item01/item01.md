## 생성자 대신 정적 팩터리 메서드를 고려하라
<div align='center'>
    <img src="img/factory_method.png" width="500px">
</div>

### [ 예시 : 물류 관리 앱의 발전 ]
`시나리오` : 물류 관리 앱을 개발하는데 첫 번째 버전은 트럭 운송만 가능해서 대부분의 코드가 트럭에 있다. 이 상황에서 앱의 사용자 수가 늘어 다양한 운송수단이 요구된다.  
`문제` : 대부분의 코드가 `Truck`에 의존하기에 `Ship` 같은 다른 운송수단을 추가하기 어렵다.

<div align='center'>
    <img src="img/factory_method_2.png" width="540px">
</div>

#### **팩터리 메서드의 등장!~**

### 팩터리 메서드
`정의` : 부모 클래스에서 객체들을 생성할 수 있는 인터페이스를 제공하지만, 자식 클래스들이 생성될 객체들의 유형을 변경할 수 있도록 하는 생성 패턴입니다.
<div align='center'>
    <img src="img/factory_method_3.png" width="540px">
</div>

위와 같은 변경 덕분에 자식 클래스에서 팩터리 메서드를 오버라이딩하고 그 메서드에 의해 생성되는 제품들의 클래스를 변경가능하다.

```
🐱영현 : 팩터리 메서드의 장점에 대해 자세히 말해주세요~
🌱상혁 : 팩터리 메서드의 장점은 여러가지가 있죠 :-)
```
### ✔️ 팩터리 메서드의 장점
#### 1) 이름을 가질 수 있다
생성자에 제공하는 파라메터가 거기서 반환하는 객체를 잘 설명하지 못할 경우에, 잘 만든 이름을 가진 static 팩토리를 사용하는 것이 사용하기 보다 더 쉽고 (해당 팩토리 메소드의 클라이언트 코드를) 읽기 편합니다. 그 예로 BigInteger.probblePrime을 들고 있다.

또, 생성자는 시그니처에 제약이 있다. 똑같은 타입을 파라미터로 받는 생성자 두개를 만들 수 없으니까 그런 경우에도 public static 팩토리 메소드를 사용하는것이 유용하다.

#### 2) 호출될 때마다 새로운 인스턴스를 만들 필요가 없다
불변(immutable) 클래스인 경우나 매번 새로운 객체를 만들 필요가 없는 경우에 미리 만들어둔 인스턴스 또는 캐시해둔 인스턴스를 반환할 수 있습니다.  
`Boolean.valueOf(boolean)` 메소드도 그 경우에 해당합니다.

#### 3) 반환 타입의 하위 타입 객체를 반환 가능하다
클래스에서 만들어 줄 객체의 클래스를 선택하는 유연함이 있습니다. 
즉, 리턴 타입의 하위 타입의 인스턴스를 만들어줘도 되니까, 리턴 타입은 인터페이스로 지정하고 그 인터페이스의 구현체는 API로 노출 시키지 않지만 그 구현체의 인스턴스를 만들어 줄 수 있다는 말입니다. 
`java.util.Collections`가 그 예에 해당한다.

`java.util.Collections`는 45개에 달하는 인터페이스의 구현체의 인스턴스를 제공하지만 그 구현체들은 전부 non-public이다. 즉 인터페이스 뒤에 감쳐줘 있고 그럼으로서 public으로 제공해야 할 API를 줄였을 뿐 아니라 **개념적인 무게(conceptual weight)**까지 줄일 수 있었다.

여기서 개념적인 무게란, 프로그래머가 어떤 인터페이스가 제공하는 API를 사용할 때 알아야 할 개념의 개수와 난이도를 말합니다.

그러한 팩토리를 사용하는 코드가 구현체가 아닌 인터페이스 타입으로 코딩하게 되는건 좋은 일~

자바 8부터 인터페이스에 public static 메소드를 추가할 수 있게 되었지만 private static 메소드는 자바 9부터 추가할 수 있습니다. 
따라서 자바 8부터 인터페이스에 public static 메소드를 사용해서 그 인터페이스의 구현체를 메소드를 제공할 수도 있지만 private static 메소드가 필요한 경우, 자바 9가 아니면 기존처럼 별도의 (인스턴스를 만들 수 없는, java.util.Collections 같은) 클래스를 사용해야 할 수도 있습니다.

#### 4) 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 가능하다
`EnumSet`의 경우 생성자 없이 정적 팩터리만 제공하는데 원소의 수에 따라 반환 클래스가 `RegularEnumSet`과 `JumboEnumSet`으로 달라집니다.
이러한 객체 타입은 외부로 노출이 되지 않기에 클라이언트는 이 두 존재를 모르며 새로운 타입을 만들거나 없애도 문제가 되지 않습니다.

#### 5) 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다
위 설명의 대표적인 예시로 `JDBC`가 있습니다.
```
public class JDBCExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mydatabase";
        String username = "root";
        String password = "password";

        // 정적 팩터리 메서드인 DriverManager.getConnection을 사용하여 Connection 객체를 생성
        try (Connection connection = DriverManager.getConnection(url, username, password)) {
            if (connection != null) {
                System.out.println("Successfully connected to the database.");
            }
        } catch (SQLException e) {
            System.err.println("Failed to connect to the database: " + e.getMessage());
        }
    }
}
```

위 예제에서 `DriverManager.getConnection(url, username, password)`는 서비스 접근 API 역할을 하는 정적 팩터리 메서드입니다.  
이 메서드는 데이터베이스 URL, 사용자 이름, 비밀번호를 받아서 Connection 객체를 반환합니다.  
여기서 주목할 점은, DriverManager는 내부적으로 적절한 Driver 클래스를 찾아서 그 클래스를 통해 Connection 객체를 생성한다는 점입니다.  
이 과정에서 Driver 클래스가 사전에 로드되어 있지 않더라도, DriverManager는 적절한 드라이버를 동적으로 로드할 수 있습니다.  
즉, 정적 팩터리 메서드를 호출하는 시점에서는 어떤 특정 Connection 구현체가 반환될지 몰라도, DriverManager는 적절한 Driver를 찾아서 Connection 객체를 생성해 줍니다.

```
🐱영현 : 그럼 팩터리 메서드는 단점이 없나요?
🌱상혁 : 물론 단점도 있습니다~
```

### ✔️ 팩터리 메서드의 단점
#### 1) 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다
정적 팩터리 메서드는 클래스의 인스턴스를 생성하기 위해 사용되는 메서드로, 보통 클래스의 생성자를 직접 호출하는 대신 사용됩니다.
```
public class EffectiveDuoClass {
    private EffectiveDuoClass() {
        // private 생성자
    }

    public static EffectiveDuoClass createInstance() {
        return new EffectiveDuoClass();
    }
}
```
위 예제에서 `EffectiveDuoClass`는 `private` 생성자를 가지고 있으며, 클래스 외부에서 직접 인스턴스를 생성할 수 없습니다.  
대신 `createInstance`라는 정적 팩터리 메서드를 통해 인스턴스를 생성할 수 있습니다.

문제는 상속을 하면서부터입니다.
```
public class SubEffectiveDuoClass extends EffectiveDuoClass {
    public SubEffectiveDuoClass() {
        super();  // 컴파일 오류: EffectiveDuoClass() has private access in EffectiveDuoClass
    }
}
```
위 코드에서 `EffectiveDuoClass`는 `EffectiveDuoClass`를 상속하려 하지만, `EffectiveDuoClass`의 생성자가 `private`이기 때문에 super()를 호출할 수 없어서 컴파일 오류가 발생합니다.

결국 이를 해결하기 위해선 `EffectiveDuoClass`의 생성자를 `public` 혹은 `protected`로 제공해야합니다.  
하지만 이렇게 되면 외부에서 직접 new 키워드를 사용하여 인스턴스를 생성하게 되므로, 메서드 이름을 통한 의도 전달이 불가능해집니다.  
즉, 팩터리 메서드의 장점인 `이름을 가지기에 반환 객체의 특성을 명확히 한다` 라는 점을 무색하게 합니다.

#### 2) 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
생성자는 Javadoc 상단에 모아서 보여주지만 static 팩토리 메소드는 API 문서에서 특별히 다뤄주지 않는다.  
정적 팩터리 메서드는 일반 메서드와 쉽게 구분되지 않기에 프로그래머가 직접 이 메서드를 찾아야합니다. 

---

### 📌 Reference

- [[Refactoring GURU] 팩토리 메서드 패턴](https://refactoring.guru/ko/design-patterns/factory-method)