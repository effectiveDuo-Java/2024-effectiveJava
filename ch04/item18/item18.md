## 상속보다는 컴포지션을 사용하라
이번 아이템에서 논하는 상속은 클래스가 다른 클래스를 확장하는 `구현 상속`을 말한다.
클래스가 인터페이스를 구현하거나 인터페이스가 다른 인터페이스를 확장하는 인터페이스 상속과는 무관하다.

### ✅ 상속의 위험성
#### 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
##### 상위 클래스가 릴리스 때 내부 구현이 달라지면, 그 여파로 하위 클래스가 오동작할 수 있다.

다음은 상속을 잘못 사용한 예이다.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}

```

여기서 getAddCount 메서드를 호출하면 3이 아니라 6이 반환된다.
이유는 hashSet의 addAll 메서드가 add를 호출해 추가하기 때문이다. 
instrumentedHashSet의 addAll은 addCount에 3을 더한 후 HashSet의 addAll 구현을 호출했다.
HashSet의 addAll은 각 원소를 add 메서드를 호출해 추가하는데, 이 때 불리는 add는 `InstrumentedHashSet`에서 오버라이딩한 add 메서드다. 따라서 addCount에
값이 중복해서 더해져, 6으로 늘어난 것이다.
addAll로 추가한 원소 하나당 2씩 늘어났다.


addAll 메서드를 다른 식으로 재정의할 수도 있다. 하지만 상위 클래스의 메서드 동작을 다시 구현하는 것은 어렵고, 
시간도 더 들고, 오류를 내거나 성능을 떨어뜨릴 수도 있다. 

##### 메서드를 재정의하는 대신 새로운 메서드를 추가하더라도, 문제가 발생할 수 있다.

1. 만약 상위 클래스에 추가된 새로운 메서드와 하위 클래스에 이미 존재하는 메서드의 이름과 시그니처가 같고, 반환 타입이 다르다면, 컴파일 오류가 발생할 수 있다.

```java
class Parent {
    // 상위 클래스에 새로운 메서드를 추가했다고 가정하자.
    public String newMethod() {
        return "Parent";
    }
}

class Child extends Parent {
    // 하위 클래스에 이미 존재하던, 반환 타입이 다른 메서드
    public int newMethod() {
        return 42;
    }
}


```

자바에서는 반환 타입만 다른 메서드 오버로딩이 허용되지 않기 때문에, 컴파일러는 어떤 메서드를 호출해야 할지 혼란스러워한다.

그러나, 시그니처와 반환 타입 모두 같을 경우라면? 상위 클래스의 메서드를 재정의한 것이 된다.

2. 새로 추가한 메서드가 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 가능성이 크다.
그 이유는 하위 클래스 작성 시 상위 클래스에 해당 메서드가 존재하지 않았기 때문에, 그 메서드가 어떤 규칙을 지켜야 하는지 전혀 알 수 없기 때문이다. 
이로 인해 기능적 충돌이나 **예기치 않은 오류**가 발생할 수 있습니다.

### ✅ 컴포지션
이러한 문제를 해결하기 위한 방법은 바로 `컴포지션`이다.
기존의 클래스를 확장하는 대신, **새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.**

새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.
이 방식을 `전달(forwarding)`이라 하며, 새 클래스의 메서드들을 `전달 메서드(forwarding method)`라 부른다.
그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

구체적인 예시를 위해 앞서 다뤘던 InstrumentedHashSet을 컴포지션과 전달 방식으로 다시 구현한 코드를 보자.
하나는 집합 클래스 자신이고, 다른 하나는 전달 메서드만으로 이뤄진 재사용 가능한 전달 클래스이다.

```java
//래퍼 클래스
public class InstrumentedHashSetUseComposition<E> extends ForwardingSet<E> {
    private int addCount = 0;

    // Set 인스턴스를 감싸고(wrap) 있다는 뜻에서 InstrumentedHashSetUseComposition 같은 클래스를 래퍼 클래스라고 한다.
    public InstrumentedHashSetUseComposition(Set<E> s) {
        super(s);
    }

    // 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴(Decorator pattern)이라고 부른다.

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}


```

이렇게

- `InstrumentedHashSetUseComposition`은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다.
- 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.


다음은 재사용할 수 있는 전달 클래스인 ForwardingSet 이다. 

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    public int size() {
        return 0;
    }

    public boolean isEmpty() {
        return s.isEmpty();
    }

    public boolean contains(Object o) {
        return s.contains(o);
    }

    public Iterator<E> iterator() {
        return s.iterator();
    }

    public Object[] toArray() {
        return s.toArray();
    }

    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    public boolean add(E e) {
        return s.add(e);
    }

    public boolean remove(Object o) {
        return s.remove(o);
    }

    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    public void clear() {
        s.clear();
    }

    @Override
    public boolean equals(Object o) {
        return s.equals(o);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s.toString();
    }
}



```

상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다. 
하지만 `컴포지션` 방식은 **한 번만 구현해두면** 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.

다만, 이 래퍼 클래스가 콜백(callback) 프레임워크와는 어울리지 않는다는 점만 주의하도록 하자.

#### 여기서 콜백 프래임워크란?
특정 이벤트가 발생했을 때 미리 지정된 메서드를 호출하는 방식으로 동작한다. 이때 메서드 호출의 대상이 되는 객체는 보통 this 키워드를 통해 전달된다.

#### 래퍼 클래스와 SELF 문제
그래서 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 되는데, 이를 `SELF` 문제라고 한다.

### 💢 상속과 is-a 
- 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.
- 즉, 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다.
- 만약 is-a 관계가 아니라면 A는 B의 필수 구성요소가 아니라 구현하는 방법 중 하나일 뿐이다.

### 💢 상속을 사용할 때는 주의해야 한다
- 컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 것과 같다.
    - 클라이언트에서 노출된 내부에 직접 접근할 수 있게 된다.
    - 최악의 경우, 클라이언트에서 상위 클래스를 직접 수정하여 하위 클래스의 불변식을 해칠 수도 있다.
- 상속은 상위 클래스의 API를 **그 결함까지도** 승계하므로 주의해야 한다.


### 📌 핵심 정리
- 상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다.
    is-a 관계일 때도 안심할 수만은 없다. 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았을 수도 있기 때문이다.
- 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용한다. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 
    `래퍼 클래스`는 하위 클래스보다 **견고하고 강력하다.**
---

### 📌 Reference
- 이펙티브 자바


