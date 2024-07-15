## 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.

- 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker{
	private static final Lexicon dictionary = ...; // 사전에 의존 

	private SpellChecker() {} // 객체 생성 방지

	public static boolean isValid(String word){...}
	public static List<String> suggestions(String typo){...}
}
```
- 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker{
	private static final Lexicon dictionary = ...; // 사전에 의존 

	private SpellChecker(...) {} 
	public static SpellChecker INSTANCE = new SpellChecker(...);

	public static boolean isValid(String word){...}
	public static List<String> suggestions(String typo){...}

}
```

### 😡 문제점
### 1) 하드코딩된 의존성
`private static final Lexicon dictionary` 필드는 특정 사전에 강하게 결합되어 있습니다.  
즉, 사전을 변경하거나 다른 사전을 사용할 수 없습니다.

### 2) 정적 메서드
정적 메서드는 다형성을 지원하지 않으며, 인터페이스를 통한 의존성 주입도 불가능합니다.  
따라서, 상속 및 확장성이 떨어지며, 테스트 시 Mocking 하기 어렵습니다.  

```java
interface Animal {
    void sound();
}

class Dog implements Animal {
    public void sound() {
        System.out.println("Bark");
    }
}

class Cat implements Animal {
    public void sound() {
        System.out.println("Meow");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myDog = new Dog();
        Animal myCat = new Cat();
        myDog.sound(); // Outputs: Bark
        myCat.sound(); // Outputs: Meow
    }
}

```  
위 예제에서 Dog와 Cat 클래스는 Animal 인터페이스를 구현하여 각각의 sound 메서드를 다르게 정의하고 있습니다.  
이는 다형성의 전형적인 예입니다.

### 🙅‍♂️ 정적 메서드의 한계 - 다형성의 부재
```java
class Animal {
    public static void sound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    public static void sound() {
        System.out.println("Bark");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myDog = new Dog();
        myDog.sound(); // Outputs: Some sound (Bark가 아니다)
    }
}
```

```
🐱영현 : 그럼 어떻게 해?
🌱상혁 : 의존 객체 주입을 사용해보자!
```  

```java
public class SpellChecker{
	private final Lexicon dictionary;

	public SpellChecker(Lexicon dictionary){
		this.dictionay = Objects.requiredNonNull(dictionay);
	}

	public boolean isValid(String word){...}
	public List<String> suggestions(String typo){...}
}
```
이를 통해 클래스의 유연성, 재사용성, 테스트 용이성을 개선해줍니다.  
또한, 불변을 보장하기에 해당 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안심하고 공유 가능합니다.

### 의존 객체 주입의 변형 - 자원 팩터리 사용
```java
public interface 상혁의생활 {
    // 쇼핑을 할 때 공통 과정
    void 먹기();
    void 자기();
    void 놀기();
    void 공부하기();
}

public abstract class 활동 {
    public 상혁의생활 생활패턴(String category){
        상혁의생활 생활패턴 = selectCategory(category);
        생활패턴.먹기();
        생활패턴.자기();
        생활패턴.공부하기();
        return 생활패턴;
    }

    // 팩터리 메서드
    abstract Shopping selectCategory(String category);
}

```

### 의존 객체 주입의 변형 - Supplier<T> 사용
```java
public class 영현 {
    private String name;

    public 영현(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        // Supplier를 사용하여 영현 객체를 생성하는 팩터리 정의
        Supplier<영현> factory = () -> new 영현("김영현");

        // 팩터리를 사용하여 영현 객체 생성
        Product 영현 = factory.get();

        // 생성된 영현 객체 사용
        System.out.println("영현 name: " + 영현.getName());
    }
}
```

의존성이 너무나 많아지는 큰 프로젝트에서는 코드를 어지럽게 만들 수도 있으니, 이럴때는 대거(Dager), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 사용하는걸 추천합니다.

---

### 📌 Reference

- 이펙티브 자바