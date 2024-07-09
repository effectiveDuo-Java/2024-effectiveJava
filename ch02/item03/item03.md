## private 생성자나 열거 타입으로 싱글턴임을 보증하라
### 싱글턴(singleton)
**싱클턴**이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 의미합니다.

### ✔️ 싱글턴의 장점
오직 하나만 생성가능하기에 재사용을 함으로써 메모리 낭비를 방지가능하며 다른 객체와 공유가 용이합니다.

### ✔️ 싱글턴의 단점
하지만 오직 하나만 생성가능하기에 테스트를 할 때 가짜 구현(mock)으로 대체 불가능합니다.

### 싱글턴 패턴 구현 방법
#### 1) final 필드

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

}
```
`public static final` 필드이기에 프로그램 실행 시 먼저 로드가 되며 이 때 `new Elvis()`를 통해 단 하나만 존재하는 인스턴스가 생성됩니다.
리플렉션을 사용하여 private 생성자를 호출하는 방법을 제외하면 Elvis는 싱글톤이 됩니다.

#### ✔️ 1-1) final 필드의 장점
이 방식이 팩터리 방식보다 명확하고 간단합니다.

#### 2) 정적 팩터리 방식
```java
public class Elvis {

    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

}
```
마찬가지로 프로그램 실행 시 `private static final` 필드이기에 먼저 로드가 된다는 공통점이 있지만 싱글톤으로 쓸 지 안쓸지를 결정가능하다는 차이점이 있습니다.
필요하다면 `Generic 싱글톤 팩터리`를 만들 수도 있습니다.
또한, static 팩터리 메서드를 `Supplier<Elvis>`에 대한 `메서드 레퍼런스`로 사용가능합니다.

```
🐱영현 : 그럼 둘 중에 어떤걸 쓰는게 좋아?
🌱상혁 : 음.. 근데 둘 다 문제가 있긴해
🐱영현 : 어떤 문제??
🌱상혁 : 직렬화랑 역직렬화 때문에 발생하는 문제들이야
```

### 🚶‍♂️ 직렬화 (Serialization)
객체를 바이트 스트림으로 변환하여 저장하거나 전송할 수 있게 만드는 과정을 의미합니다.
```java
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("object.data"));
out.writeObject(someObject);
out.close();
```

### 🚶‍♂️‍➡️ 역직렬화 (Deserialization)
저장된 바이트 스트림을 읽어 다시 객체로 변환하는 과정입니다.
```java
ObjectInputStream in = new ObjectInputStream(new FileInputStream("object.data"));
SomeClass someObject = (SomeClass) in.readObject();
in.close();
```

위에서 언급한 final 필드 방식과 정적 팩터리 방식 모두 직렬화/역직렬화 시 새로운 인스턴스가 발생합니다.
```java
Singleton instance1 = Singleton.INSTANCE;
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("singleton.data"));
out.writeObject(instance1);
out.close();

ObjectInputStream in = new ObjectInputStream(new FileInputStream("singleton.data"));
Singleton instance2 = (Singleton) in.readObject();
in.close();

System.out.println(instance1 == instance2); // false
```

```
🐱영현 : 뭐야 그럼 둘 다 문제덩어리네?
🌱상혁 : 걱정마. readResolve 메서드를 사용하면 다~ 해결돼
```  

싱글턴 클래스에 readResolve 메서드를 추가하면 역직렬화 시 자동 호출이 되며, 이 메서드가 반환하는 객체가 실제로 역직렬화 객체 대신 사용됩니다. 
```java
public class Singleton implements Serializable {
    public static final Singleton INSTANCE = new Singleton();

    private Singleton() {
        // private constructor
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```
```java
Singleton instance1 = Singleton.INSTANCE;
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("singleton.data"));
out.writeObject(instance1);
out.close();

ObjectInputStream in = new ObjectInputStream(new FileInputStream("singleton.data"));
Singleton instance2 = (Singleton) in.readObject();
in.close();

System.out.println(instance1 == instance2); // true
```

```
🐱영현 : 아.. 좀 귀찮은데
🌱상혁 : (세상에..) 그럼 열거 타입 방식의 싱글턴을 사용해보는건 어때?
```  
### 열거 타입 방식의 싱글턴
```java
public enum Elvis {
    INSTANCE;
}
```
직렬화/역직렬화 할 때 코딩으로 문제를 해결할 필요도 없고, 리플렉션으로 호출되는 문제도 고민할 필요없습니다.
다만, Enum 말고 다른 상위 클래스를 상속해야 한다면 사용할 수 없습니다.

---

### 📌 Reference

- 이펙티브 자바
- [[Inpa Dev] 자바 직렬화(Serializable) - 완벽 마스터하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)