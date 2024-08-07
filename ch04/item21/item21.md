## 인터페이스는 구현하는 쪽을 생각해 설계하라
### 🔙 Java 8 이전
**현재 인터페이스에 새로운 메서드가 추가될 일은 영원히 없다** 라는 생각으로 클래스를 작성했습니다.  
따라서, 기존 구현체를 깨트리지 않고 인터페이스에 메서드를 추가할 방법이 없었습니다.  

Java 8 이전의 인터페이스는 이러한 생각과 함께 인터페이스를 implement 한 클래스는 사실상 디폴드 메서드의 존재조차 모른채 삽입됩니다.

### 🔜 Java 8 이후
**디폴트 메서드**가 등장했습니다.  
디폴드 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴드 메서드를 재정의하지 않은 모든 클래스에서 디폴드 구현이 사용됩니다.  

- 디폴드 메서드를 포함하는 인터페이스
```java
interface DuoInterface {
    void abstractMethod();

    default void defaultMethod() {
        System.out.println("디폴드 메서드 구현.");
    }
}
```

- 디폴드 메서드를 재정의 하지 않은 클래스
```java
class ClassWithoutOverride implements DuoInterface {
    @Override
    public void abstractMethod() {
        System.out.println("ClassWithoutOverride에서의 추상 메서드 구현.");
    }
}
```  

- 디폴드 메서드를 재정의 한 클래스
```java
class ClassWithOverride implements DuoInterface {
    @Override
    public void abstractMethod() {
        System.out.println("ClassWithOverride에서의 추상 메서드 구현.");
    }

    @Override
    public void defaultMethod() {
        System.out.println("ClassWithOverride에서의 default 메서드 구현.");
    }
}
```

- Main 클래스
```java
public class Main {
    public static void main(String[] args) {
        DuoInterface instance1 = new ClassWithoutOverride();
        DuoInterface instance2 = new ClassWithOverride();

        instance1.abstractMethod(); // 클래스에서 구현한 추상 메서드
        instance1.defaultMethod();  // 인터페이스의 디폴트 메서드 사용

        instance2.abstractMethod(); // 클래스에서 구현한 추상 메서드
        instance2.defaultMethod();  // 클래스에서 재정의한 디폴트 메서드 사용
    }
}
```  

- 출력 결과
```
ClassWithoutOverride에서의 추상 메서드 구현.
디폴드 메서드 구현.

ClassWithOverride에서의 추상 메서드 구현.
ClassWithOverride에서의 디폴드 메서드 구현.
```  

`ClassWithoutOverride`는 `defaultMethod`를 재정의하지 않았기 때문에 `DuoInterface`에서 제공하는 기본 구현을 사용합니다.  
`ClassWithOverride`는 `defaultMethod`를 재정의했기 때문에 자신의 구현을 사용합니다.  

### Java 8에서 Collection에 추가된 removeIf 디폴드 메서드
```java
default boolean removeIf(Predicate<? super E> filter) {
  Objects.requireNonNull(filter);
  boolean result = false;
  for (Iterator<E> it = iterator(); it.hasNext(); ) {
    if (filter.test(it.next())) {
        it.remove();
        result = true;
    }
  }
  return true;
}
```

```
🐱영현 : Multi-Thread인 경우는 어떻게 돼?
🌱상혁 : 에러가 날 수 있지
🐱영현 : 그럼 어떻게 해?
🌱상혁 : 재정의 하면 돼 :-)
```  

```java
@Override
public boolean removeIf(final Predicate<? super E> filter) {
    synchronized(lock) {
        return decorated().removeIf(filter);
    }
}
```  

```
🐱영현 : 웬 public?
🌱상혁 : 인터페이스 메소드를 Override 할 시 반드시 public으로 해야돼!
```  

Java 플랫폼에 속하지 않는 제3의 기존 컬렉션 구현체들(Apache의 synchronizedCollection)은 이런 언어 차원의 인터페이스 변화에 발맞춰 수정될 기회가 없었으며, 그 중 일부는 여전히 수정되지 않고 있다!

### 📌 결론
디폴트 메소드는 런타임 오류를 일으킬 수 있다.  
인터페이스를 설계할 때는 세심한 주의를 기울이고 인터페이스의 경우 릴리즈 전 반드시 테스트를 거쳐야 한다. 

---

### 📌 Reference
- 이펙티브 자바
- 윤성우의 열혈 Java 프로그래밍

