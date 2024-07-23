## clone 재정의는 주의해서 진행해라

### 객체 복사
#### clone()은 사용하기에 몇가지 문제점이 있다.
- clone() 메서드가 선언된 곳이 Cloneable이 아닌 Object이다.
- clone() 메서드의 접근 제한자가 protected 이다.
- Cloneable을 구현하는 것만으로는 외부 객체에서 clone()를 호출할 수 없다.

이러한 문제에도 불구하고 `Cloneable` 방식은 널리 쓰이고 있어서 잘 알아두는 것이 좋다.

#### ❗ 이 아이템에서 다룰 내용 ❗
- clone() 메서드를 잘 동작할 수 있는 구현 방법
- clone()을 사용하기 위한 상황
- 그 외 대체 방법

### cloneable 인터페이스 용도
- Object의 protected 메서드인 clone의 동작 방식을 결정한다.
- Cloneable 을 구현한 클래스의 인터페이스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환한다.
- Cloneable 을 구현하지 **않은** 클래스에서 clone을 호출하는 경우 `CloneNotSupportedException`을 던진다.

##### 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공한다.

### clone 메서드의 허술한 일반 규약
- 복사의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.
- 일반적인 의도에 따르면
	- 아래 내용에 대한 요구가 반드시 참은 아니다.
		```java
		// 복사본과 원본이 일치하지 않는다.
		x.clone() != x

		// 클래스 인스턴스 타입은 같다.
		x.clone().getClass() == x.getClass()
		```
	- 아래 일반 식도 일반적으로는 참이지만, 필수는 아니다.
		```java
		// 동치성
		x.clone().equals(x)
		```
	- 관례상, clone()는 super.clone()을 호출해서 반환된 객체를 반환한다.
		```java
		x.clone().getClass() == x.getClass()
		```
	- 관례상, 반환된 객체와 원본 객체는 독립적이다.
	- 이를 만족하려면 super.clone() 으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수 있다.

### clone() 재정의시 주의사항 - 1
- clone 메서드가 super.clone()이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 동일한 인스턴스로 본다.
- 이 클래스의 하위 클래스에서 super.clone()을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone() 메서드가 제대로 동작하지 않게 된다.
	- 상위 클래스의 clone() 값을 반환하면 안되고, 하위 클래스 타입으로 변환한 뒤 반환 해야 한다.
	- clone() 을 재정의한 클래스가 final 클래스인 경우에는 걱정할 필요가 없다.
- 모든 필드가 기본 타입이거나 불변 객체를 참조하는 경우 이 객체는 완벽한 상태이므로 clone()를 제공하지 않는 것이 좋다.
	- 쓸데 없는 복사를 지양한다는 관점에서 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋다.

이 점들을 고려하여 PhoneNumber의 clone 메서드는 다음처럼 구현이 가능하다.
```java
@Override public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
이렇게 Object의 clone 메서드는 Object를 반환하지만 PhoneNumber의 clone 메서드는 `PhoneNumber`를 반환하게 했다.
이렇게 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.
이 방식으로 클라이언트가 형변환 하지 않아도 되게끔 해준다.
try-catch 블록으로 Object의 clone 메서드가 `검사 예외(checked exception)` 로 제공되는 것을 `비검사 예외(unchecked exception)` 로 처리하도록 한다.

### clone() 재정의시 주의사항 - 2) 가변 객체를 참조하는 경우
- clone 메서드가 단순히 super.clone() 의 결과를 그대로 받아오는 경우?
	- 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.
	- 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 말이다.
	- 따라서 프로그램이 이상하게 동작하거나 NullPointerException 이 발생하게 된다.
- clone 메서드는 사실상 생성자와 같은 효과를 낸다.
	- 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

	//super.clone()의 결과를 그대로 받아오는 경우
	 @Override
    public Stack clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

```

##### - 가변 객체를 갖는 클래스를 복제하기 위해서는 내부 정보를 복사해야 함!
가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출하는 것이다.

```java
    @Override
    public Stack clone() {
        try {
            Stack clone = (Stack) super.clone();
            // 배열의 clone은 런타임 타입과 컴파일 타입 모두가 원본 배열과 같은 배열을 반환한다.
            // 따라서 배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장한다.
            // 배열은 clone 기능을 제대로 사용하는 유일한 예이다.
            clone.elements = elements.clone();
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
```
💡 한편, elements 필드가 final이었다면 위의 방식은 동작하지 않는다!
final 필드에서는 새로운 값을 할당할 수 없기 때문이다.
이는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다. (단, 원본과 복제된 객체가 그 가변 객체를 공유해도 안전하다면 괜찮다)
그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.
```

### clone() 재정의시 주의사항 - 3) 복잡한 가변 객체를 참조하는 경우
배열에서처럼 clone을 재귀적으로 호출하는 것만으로 충분하지 않을 때도 있다.

이번에는 해시테이블용 clone 메서드를 생각해보자.
해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조한다.

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    // 나머지 코드는 생략
}

```

#### ❌ 잘못된 clone 메서드
```java
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    } catch(CloneNotSupportedException e) {
        throw new Assertion();
    }
}
```
Stack에서처럼 단순히 버킷 배열의 clone을 재귀적으로 호출하였다.
하지만 이 배열은, 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다.

**따라서, 각 버킷을 구성하는 연결 리스트를 복사해야 한다.**

#### 💡 재귀적 clone 메서드

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    private static class Entry {
        ...
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    
    // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
    Entry deepCopy(){
        return new Entry(key,value,
            next==null ? null : next.deepCopy());
    }
	
    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i =0; i < buckets.length; i++){
                if(buckets[i] != null){
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
    
    // 나머지 코드는 생략
}

```

Entry가 깊은 복사(deep copy)를 지원하도록 보강되었다.
HashTable의 clone 메서드는
- 적절한 크기의 새로운 버킷 배열 할당한다.
- 원래 버킷 배열을 순회하면서, 비어있지 않은 각 버킷에 대해 깊은 복사를 수행한다. (deepCopy 메서드)
- 여기서 deepCopy 메서드는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다.
이 기법은 간단하고 잘 작동하지만, 리스트가 길어지면 스택 오버플로우를 일으킬 위험이 있기 때문에 연결 리스트를 복제하는 방법으로는 좋지 않다.

#### 💡 반복적으로 연결리스트를 복사하는 deepCopy 메서드
이러한 재귀 호출의 문제를 해결하기 위해 반복문을 써서 순회한다.

```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```
#### 고수준 메서드
이제 복잡한 가변 객체를 복제하는 마지막 방법을 살펴보자.

1. 먼저 super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한다.
2. 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다.

`HashTable` 예에서라면,
1. buckets 필드를 새로운 버킷 배열로 초기화하하고
2. 원본 테이블에 담긴 모든 키-값 쌍 각각에 대해 복제본 테이블의 `put(key, value)` 메서드를 호출한다.
고수준 API를 활용해 복제하면 우아한 코드를 얻게 되지만, 저수준에서 바로 처리할 때보다는 느리다.

또한, clone 메서드는 생성자와 마찬가지로 재정의될 수 있는 메서드를 호출하지 않아야 한다. 
만약 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다.

따라서, 위에 말한 `put(key, value)`는 final이거나 private이어야 한다.

### (+) clone 메서드 주의 사항

#### 1. public인 clone 메서드에서는 throws 절을 없애야 한다.
검사 예외를 던지지 않아야 그 메서드를 사용하기 편하기 때문이다. 

#### 2. 상속용 클래스는 Cloneable을 구현해서는 안 된다.
clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수 있다.
다음과 같이 clone을 퇴화시켜놓으면 된다.

```java
@Override
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```
이는 하위 클래스에서 cloneable을 지원하지 못하게 하는 clone 메서드이다.

#### 3. Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야 한다. 
Object의 clone 메서드는 동기화를 신경쓰지 않았다.
그러니 super.clone 호출 외에 다른 할 일이 없더라도 clone을 재정의하고 동기화해줘야 한다.

### 🔻 복사 생성자와 복사 팩터리 (또 다른 객체 복사 방식)
Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 `clone*을 잘 작동하도록 구현해야 한다.
그러나 그렇지 않은 상황에서는 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.

#### 💡 복사 생성자
복사 생성자란, 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.
```java
public Yum(Yum yum) { ... }
```

#### 💡 복사 팩터리
복사 팩터리는 복사 생성자를 모방한 정적 팩터리다.
```java
public static Yum newInstance(Yum yum) { ... }
```

이 둘의 방식이 Cloneable/clone 방식보다 뛰어난 점은 다음과 같다.

1. 언어 모순적이고 위험천만한 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않는다.
2. 엉성하게 문서화된 규약에 기대지 않는다.
3. 정상적인 final 필드 용법과도 충돌하지 않는다.
4. 불필요한 검사 예외를 던지지 않는다.
5. 형변환이 필요하지 않다.
6. 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
	변환 생성자, 변환 팩터리로써, 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.


### 📌 핵심 정리
새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다.
final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만,
성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다.

따라서
기본 원칙은 **'복제 기능은 생성자와 팩터리를 이용하는 게 최고'** 라는 것이다.
단 **배열**만은 `clone 메서드 방식`이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.

---

### 📌 Reference
- 이펙티브 자바
