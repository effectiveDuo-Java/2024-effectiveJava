## 다 쓴 객체 참조를 해제하라  
C나 C++과는 달리 Java는 JVM의 GC가 메모리를 관리해줍니다.  

```
🐱영현 : 그럼 관리 안해야징~
🌱상혁 : 꾸짖을 갈(喝)!!!!
```  

자바가 알아서 다 쓴 객체를 회수한다고 해서 메모리 관리에 신경을 쓰지 않으면 안됩니다.  

```java
public class Stack {
	private static final int DEFAULT_INITAL_CAPACITY = 16;

	private Obejct[] elements;
	private int size = 0;
	
	public Stack() {
		elements = new Object[DEFAULT_INITAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++];
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		return elements[--size];
	}

	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
}
```  
위 예제 코드를 보았을 때 문제가 없는것 같지만, 메모리 누수 문제가 발생하고 있습니다.  
`elements[--size]`를 통해 스택에서 꺼내진 객체들은 GC입장에선 참조가 되는지 안되는지 모르기에 회수하지 못할 수 있습니다.  
다 쓴 참조(obsolete reference)를 가지고 있다는 점에서 메모리 누수가 발생합니다.  

### ✔️ 해결 방안
### 🚫 null 처리  
해당 참조를 다 사용했을 때 null 처리(참조 해제)하면 됩니다.

```java
public Object pop() {
	if (size == 0) {
		return new EmptyStackException();
	}

	Object result = elements[--size];
	elements[size] = null; // 다 쓴 참조 해제
	return result;
}
```  
또한, null 처리한 참조를 실수로 사용하고자 한다면 `NullPointerException`을 던지며 종료됩니다.  

```
🐱영현 : 귀찮게 모든 참조에 null 처리를 해줘야해?
🌱상혁 : 그건 또 아니야. 예외적인 경우에만 처리를 해.
```  

### 1) null 처리 - 메모리를 직접 관리할 때  🧑‍💻 
앞선 Stack 코드의 문제점은 배열의 활성 영역만 사용되지만 GC 입장에선 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체로 보입니다.  
따라서, 비활성 영역이 되는 순간 개발자는 null 처리를 해서 GC에게 알려야합니다.  

### 2) null 처리 - 캐시⚡  
엔트리가 살아 있는 캐시가 필요한 상황이라면 WeakHashMap을 사용하자!  

WeakHashMap에 사용되는 WeakReference부터 알아보겠습니다.

```java
public class WeakReferenceExample {
    public static void main(String[] args) {
        SomeObject obj = new SomeObject("난 강한 참조!!");
        WeakReference<SomeObject> weakRef = new WeakReference<>(obj);

        System.out.println("null 처리 전 강한 참조: " + weakRef.get());

        // 강한 참조 null 처리
        obj = null;

        // GC 요청
        System.gc();

        // GC 에게 실행 시간 할당
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("강한 참조 null 처리 및 GC 후: " + weakRef.get());
    }
}

class SomeObject {
    private String name;

    public SomeObject(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return name;
    }
}
```  
강한 참조가 null로 설정되면 가비지 컬렉션에 의해 약한 참조된 객체가 수거될 수 있습니다.  

```java
public class WeakHashMapExample {
    public static void main(String[] args) {
        Map<SomeObject, String> map = new WeakHashMap<>();

        SomeObject key1 = new SomeObject("Key1");
        SomeObject key2 = new SomeObject("Key2");

        map.put(key1, "Value1");
        map.put(key2, "Value2");

        System.out.println("null 처리 전 key: " + map);

        // key1에 대한 강한 참조 null 처리
        key1 = null;

        // GC 요청
        System.gc();

        // GC 에게 실행 시간 할당
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("key1에 대한 강한 참조 null 처리 및 GC 후: " + map);
    }
}

class SomeObject {
    private String name;

    public SomeObject(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return name;
    }

    @Override
    public int hashCode() {
        return name.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        SomeObject that = (SomeObject) obj;
        return name.equals(that.name);
    }
}
```  
key1을 null로 설정한 후 가비지 컬렉션을 요청하면, key1에 해당하는 엔트리가 자동으로 제거됩니다.

### 3) null 처리 - 리스너(listener), 콜백(callback) 📞
클라이언트가 콜백을 등록만 하고 해지하지 않는다면 계속 쌓여갑니다.  
이럴 때 콜백을 약한 참조로 저장하면 GC가 수거해갑니다.

---

### 📌 Reference

- 이펙티브 자바