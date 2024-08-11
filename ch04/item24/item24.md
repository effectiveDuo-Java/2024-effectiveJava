## 멤버 클래스는 되도록 static으로 만들라
### 중첩 클래스(nested class)
중첩 클래스란 다른 클래스 안에 정의된 클래스를 말합니다.  
중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외 쓰임새가 있다면 톱레벨 클래스로 만들어야 합니다.  

> **톱레벨 클래스(TopLevel Class)란?**  
다른 클래스나 인터페이스 내부에 선언되지 않은 클래스입니다. 즉, 파일의 최상위 레벨에 위치한 클래스를 의미합니다.  
자바 소스 파일 내에서 독립적으로 존재하며, 중첩 클래스와 구분됩니다.  

### 중첩 클래스 종류
### 🛑 정적 멤버 클래스  

```java
class Caculator {
    // 열거 타입도 정적 멤버 클래스 
    public enum Operation {
        PLUS, MINUS
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Calculator.Operation.PLUS); // PLUS
    }
}
```  

private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분을 나타낼 때 사용합니다.  

<br></br>

### 🟢 비정적 멤버 클래스  
정적 멤버 클래스와 차이점
구문상: `static` 여부 
의미상: 바깥 클래스의 인스턴스와 연결되어 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 `this`를 사용해 바깥 인스턴스 메서드를 호출하거나 참조를 가져올 수 있다.  

> **정규화된 this**:   
클래스명.this 형태로 자바에서 멤버 클래스(내부 클래스)가 바깥 클래스의 인스턴스와 연결되어 있을 때, 내부 클래스에서 바깥 클래스의 인스턴스를 명확하게 가리키기 위해 사용하는 표현입니다.  

```java
public class OuterClass {

    private String message = "안녕! 난 바깥클래스야.";

    // 바깥 클래스의 메서드에서 비정적 멤버 클래스의 인스턴스 생성
    public void createInnerInstance() {
        InnerClass inner = new InnerClass();  // 내부 클래스의 인스턴스를 생성
        inner.displayMessage();
    }

    public class InnerClass {

        // 내부 클래스의 생성자
        public InnerClass() {
            System.out.println("내부 클래스 인스턴스 생성.");
        }

        // 내부 클래스의 메서드에서 바깥 클래스의 인스턴스에 접근
        public void displayMessage() {
            System.out.println("바깥 클래스로부터의 메세지: " + message);
        }
    }

    public static void main(String[] args) {
        // 바깥 클래스의 인스턴스를 생성
        OuterClass outer = new OuterClass();

        // 바깥 클래스의 인스턴스 메서드를 호출해 내부 클래스의 인스턴스를 생성
        outer.createInnerInstance();

        // 다른 바깥 클래스 인스턴스 생성
        OuterClass anotherOuter = new OuterClass();

        // 두 번째 바깥 클래스 인스턴스에서 내부 클래스 인스턴스 생성
        InnerClass anotherInner = anotherOuter.new InnerClass();
        anotherInner.displayMessage();
    }
}
```
```
내부 클래스 인스턴스 생성.
바깥 클래스로부터의 메세지: 안녕! 난 바깥클래스야.
내부 클래스 인스턴스 생성.
바깥 클래스로부터의 메세지: 안녕! 난 바깥클래스야.
```

비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스(내부 클래스)가 인스턴스화 될 때 확립되며 이는 변경할 수 없습니다.  
일반적으로 위와 같이 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어집니다.  
하지만, 드물게도 직접 `바깥 인스턴스의 클래스.new MemberClass(args)`를 호출해 수동으로 만들기도 합니다.  

<br></br>

```java
public class OuterClass {

    private String message = "안녕! 난 바깥클래스야.";

    public void outerMethod() {
        System.out.println("바깥 클래스 메서드 호출");
    }

    public class InnerClass {

        public void innerMethod() {
            // 바깥 클래스의 인스턴스 메서드를 호출
            OuterClass.this.outerMethod();

            // 바깥 클래스의 인스턴스 변수에 접근
            System.out.println("바깥 클래스로부터의 메세지: " + OuterClass.this.message);
        }
    }

    public static void main(String[] args) {
        // 바깥 클래스의 인스턴스 생성
        OuterClass outer = new OuterClass();

        // 비정적 멤버 클래스의 인스턴스 생성
        OuterClass.InnerClass inner = outer.new InnerClass();

        // 내부 클래스의 메서드 호출
        inner.innerMethod();
    }
}
```   
```
바깥 클래스 메서드 호출
바깥 클래스로부터의 메세지: 안녕! 난 바깥클래스야.
```  

### 비정적 클래스의 사용
- 비정적 멤버 클래스는 `어댑터 패턴(Adapter Pattern)`을 정의할 때 자주 쓰인다.  
- 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용
- Map 인터페이스의 구현체에서 자신의 컬렉션 뷰를 구현할 때 사용
- `Set`와 `List`같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때 비정적 멤버 클래스를 주로 사용  

<details>
<summary>컬렉션 뷰란?</summary>

### 컬렉션 뷰란?
컬렉션 뷰는 다른 객체의 데이터를 별도로 복사하거나 저장하지 않고, 해당 객체의 데이터를 그대로 보여주는 방식입니다. 즉, 컬렉션 뷰는 자체 데이터를 갖고 있지 않고, 모든 작업이 다른 객체에 있는 데이터를 바탕으로 이루어집니다.

### Map과 keySet()
Map의 keySet() 메서드를 예로 들어보겠습니다.  
keySet()은 Map의 키들만을 보여주는 Set 뷰를 반환합니다.  
이 Set 뷰는 실제로 Map의 키 데이터를 따로 저장하지 않고, Map의 데이터를 바탕으로 동작합니다.

```java
class KeySet implements Set<K> {
  private final Map<K, V> map;

  public boolean contains(Object o) {
    return map.containsKey(o);
  }

  ...
}
```  

이 코드에서 보듯이, KeySet 클래스는 Map 객체를 참조하고, contains 메서드 호출 시 Map의 containsKey 메서드를 사용합니다.

즉, keySet() 뷰는 Map과 연결되어 있어서 Map의 데이터를 실시간으로 반영합니다. 예를 들어, Map에서 어떤 키를 제거하면, keySet에서도 그 키가 자동으로 사라집니다. 따로 keySet을 업데이트할 필요가 없습니다.

### Arrays.asList(array)  
다른 예로, Arrays.asList(array)는 배열을 List로 보여주는 뷰를 제공합니다. 이 List는 배열의 데이터를 따로 복사하지 않고, 배열을 그대로 이용합니다.

```java
String[] strings = {"a", "b", "c"};
List<String> list = Arrays.asList(strings);
System.out.println(list.get(0)); // "a"
strings[0] = "d";
System.out.println(list.get(0)); // "d"
list.set(0, "e");
System.out.println(strings[0]); // "e"
```  

이 예시에서, 배열 strings의 첫 번째 값을 바꾸면, list에서도 그 값이 바뀝니다.  
반대로, list에서 값을 바꾸면 배열에서도 값이 바뀝니다.  
이는 Arrays.asList 뷰가 배열의 데이터를 복사하지 않고, 원래 배열에 있는 데이터를 그대로 참조하기 때문에 발생합니다.

### 장점
컬렉션 뷰를 사용하는 주요 장점은 효율성입니다. 예를 들어, 배열을 List로 변환할 때 새로운 ArrayList를 생성하고 데이터를 복사하는 대신, Arrays.asList는 추가적인 메모리 사용 없이 원래 배열을 참조하여 List 메서드를 구현합니다. 이렇게 하면 메모리 사용량이 최소화되고, 불필요한 데이터 복사가 방지됩니다.

### 요약
컬렉션 뷰는 원본 데이터에 대한 다른 관점을 제공하는 방법입니다. 이 방식은 데이터를 복사하지 않기 때문에 메모리 효율적이며, 원본 데이터의 변경 사항이 즉시 뷰에도 반영되는 특성을 가지고 있습니다.
</details>

```java
public class MySet<E> extends AbstractSet<E>{
    ... //생략

    @Override public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        ...
    }
}
```  

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들어야 합니다.**  
`static`을 생략하면 바깥 인스턴스로부터 숨은 외부 참조를 가지게 되는데 이 때 참조를 저장하기 위해 시간과 공간이 소비됩니다.  
가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못해 메모리 누수가 발생할 수도 있습니다.  

```java
public class OuterClass {
    private String outerField = "Outer field";

    // InnerClass는 비정적 멤버 클래스입니다.
    public class InnerClass {
        public void printOuterField() {
            System.out.println(outerField); // 외부 클래스의 필드에 접근
        }
    }
}

// 사용 예시
OuterClass outer = new OuterClass();
OuterClass.InnerClass inner = outer.new InnerClass();
// outer 인스턴스를 사용하지 않더라도 inner 인스턴스가 outer를 참조하고 있습니다.
```  

위 예시에서 InnerClass는 비정적 멤버 클래스입니다. InnerClass의 인스턴스 inner는 outer 인스턴스를 숨은 참조로 가지고 있습니다. 따라서 outer가 더 이상 필요 없어져도, inner가 outer를 참조하는 한 outer는 가비지 컬렉터에 의해 수거되지 않습니다.  

<br></br>

Map 인스턴스의 경우 Map 구현체는 `엔트리(Entry)` 객체들을 가지고 있습니다.  
모든 엔트리가 Map과 연관되어 있지만 엔트리의 getKey, getValue, setValue는 Map을 직접 사용하지 않습니다.  
즉, 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비입니다.  

- 올바른 접근: Entry를 정적 멤버 클래스로 선언
```java
import java.util.HashMap;
import java.util.Map;

public class CustomMap<K, V> {

    private Map<K, V> map = new HashMap<>();

    // 정적 멤버 클래스로 Entry 선언 (올바른 방법)
    private static class Entry<K, V> implements Map.Entry<K, V> {
        private final K key;
        private V value;

        public Entry(K key, V value) {
            this.key = key;
            this.value = value;
        }

        @Override
        public K getKey() {
            return key;
        }

        @Override
        public V getValue() {
            return value;
        }

        @Override
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }
    }

    public void put(K key, V value) {
        map.put(key, value);
        Entry<K, V> entry = new Entry<>(key, value);
        // 예를 들어, entry 객체를 어디엔가 저장할 수 있음
    }

    public V get(K key) {
        return map.get(key);
    }

    public static void main(String[] args) {
        CustomMap<String, Integer> customMap = new CustomMap<>();
        customMap.put("key1", 1);
        customMap.put("key2", 2);

        System.out.println(customMap.get("key1")); // 1
        System.out.println(customMap.get("key2")); // 2
    }
}
```  
Entry 클래스는 CustomMap의 인스턴스에 종속되지 않기 때문에, Entry 객체는 바깥 CustomMap 객체를 참조하지 않습니다.  
이는 메모리 효율적이며 성능에도 유리합니다.

- 잘못된 접근: Entry를 비정적 멤버 클래스로 선언  
```java
import java.util.HashMap;
import java.util.Map;

public class CustomMap<K, V> {

    private Map<K, V> map = new HashMap<>();

    // 비정적 멤버 클래스로 Entry 선언 (잘못된 방법)
    private class Entry implements Map.Entry<K, V> {
        private final K key;
        private V value;

        public Entry(K key, V value) {
            this.key = key;
            this.value = value;
        }

        @Override
        public K getKey() {
            return key;
        }

        @Override
        public V getValue() {
            return value;
        }

        @Override
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }
    }

    public void put(K key, V value) {
        map.put(key, value);
        Entry entry = new Entry(key, value);
        // 비정적 멤버 클래스 Entry 객체는 CustomMap 인스턴스를 참조합니다.
    }

    public V get(K key) {
        return map.get(key);
    }

    public static void main(String[] args) {
        CustomMap<String, Integer> customMap = new CustomMap<>();
        customMap.put("key1", 1);
        customMap.put("key2", 2);

        System.out.println(customMap.get("key1")); // 1
        System.out.println(customMap.get("key2")); // 2
    }
}
```  
Entry 객체는 실제로 CustomMap 객체를 참조할 필요가 없습니다.  
Entry의 메서드(getKey, getValue, setValue)는 모두 키와 값에만 접근할 뿐, Map 객체와의 상호작용은 하지 않습니다.  
그러나 비정적 멤버 클래스로 선언된 경우, Entry 객체는 무조건 CustomMap 인스턴스를 참조하게 되어 메모리 낭비가 발생합니다.


### 익명 클래스  
#### 익명 클래스의 특징
- 이름이 없다.
- 바깥 클래스의 멤버가 아니다.  
- 쓰이는 시점에 선언과 동시에 인스턴스가 생성된다.  
- 코드의 어디서든 생성 가능하며, 오직 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조 가능하다.  
- 상수 정적 변수 외에는 정적 변수를 가질 수 없다.  

```java
public class AnonymousExample {
    private double x;
    private double y;

    public double operate() {
        Operator operator = new Operator() {
            @Override
            public double plus() {
                System.out.printf("%f + %f = %f\n", x, y, x + y);
                return x + y;
            }
            @Override
            public double minus() {
                System.out.printf("%f - %f = %f\n", x, y, x - y);
                return x - y;
            }
        };
        return operator.plus();
    }
}

interface Operator {
    double plus();
    double minus();
}
```  
#### 익명 클래스의 제약 사항  
- 선언한 지점에만 인스턴스를 만들 수 있다.
- instanceof 검사나 클래스 이름이 필요한 작업은 수행 불가능
- 여러 인터페이스 구현 불가능
- 인터페이스를 구현하는 동시에 다른 클래스 상속 불가능
- 클라이언트는 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출 불가능
- 표현식 중간에 있어, 코드가 긴 경우 가독성이 떨어진다.

```java
static List<Integer> intArrayAsList(int[] a){
    Objects.requireNonNull(a);

    return new AbstractList<>(){

        // AbstractList의 abstract 메서드로 반드시 구현해야함
        @Override public Integer get(int i){
            return a[i]; 
        }

        // 선택적으로 구현
        @Override public Integer set(int i,Integer val){
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }

        // AbstractCollection의 abstract 메서드로 반드시 구현해야함
        @Override public int size(){
            return a.length; 
        }
    }
}
```

### 지역 클래스  
중첩 클래스 중 가장 드물게 사용된다.
- 지역 클래스는 지역 변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언할 수 있고, 유효 범위도 지역변수와 같다. 
- 멤버 클래스 처럼 이름이 있으며, 반복해서 사용할 수 있다.
- 익명 클래스처럼 비정적 문맥에서 사용될 떄만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버는 가질 수 없고, 가독성을 위해 짧게 작성해야한다.

```java
public class LocalClassExample {

    public void calculateSum() {
        int x = 10;
        int y = 20;

        // 지역 클래스 선언
        class Adder {
            public int add() {
                return x + y;
            }
        }

        // 지역 클래스 인스턴스 생성 및 사용
        Adder adder = new Adder();
        int sum = adder.add();
        System.out.println("Sum: " + sum);
    }

    public static void main(String[] args) {
        LocalClassExample example = new LocalClassExample();
        example.calculateSum(); // Sum: 30
    }
}

```

### 📌 핵심 정리
- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다.  
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조하는 경우에는 비정적으로, 그렇지 않다면 정적으로 구현해라.  
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고, 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있는 경우 익명 클래스로, 그렇지 않다면 지역 클래스로 구현해라.  

---

### 📌 Reference
- 이펙티브 자바