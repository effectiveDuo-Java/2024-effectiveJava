## Comparable을 구현할지 고려하라

### compareTo
이번에는 Compareable 인터페이스의 유일한 메서드인 `compareTo`에 대해 알아보자.`compareTo`는 다른 메서드와 달리 Object의 메서드가 아니다.
`compareTo`는 아래 두가지만 빼면 Object의 equals와 같다.
1. compareTo 는 단순 동치성 비교에 더해 순서까지 비교할 수 있다.
2. compareTo 는 제네릭하다.

또한, 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 할 수 있다.
String이 Comparable을 구현하였기 때문에 다음 예제는 명령줄 인수들을 (중복 제거하고) 알파벳순으로 출력한다.
사실상 자바 플랙폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현했다.

```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

그러니, **순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.**

### ✅ compareTo 메서드의 일반 규약
equals의 규약과 비슷하다.

이 객체와 주어진 객체의 순서를 비교한다.
이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.
이 객체와 비교할 수 없는 타입의 객체가 주어지면 `ClassCastException`을 던진다.

sgn: 부호 함수(signum function), 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환한다.

1. sgn(x.compareTo(y)) = -sgn(y.compareTo(x))
y.compareTo(x)가 Exception을 발생시킨다면
x.compareTo(y) 또한 Exception을 발생시켜야 한다.
-> **반사성**
2. (x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면 x.compareTo(z) > 0
→ **추이성**
3. x.compareTo(y) == 0 이면 x.compareTo(z) == y.compareTo(z)
크기가 같은 객체들끼리는 항상 같아야 한다.
-> **대칭성**

위의 3가지 규약은 compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 **반사성**, **대칭성**, **추이성**을 충족해야 함을 뜻한다.

4. 필수는 아니지만 꼭 지키는 게 좋다.
    (x.compareTo(y) == 0) == (x.equals(y)) 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다.
    "주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다."
-> compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 것이다.
이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관되게 된다.
정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용함에 주의하자!

### ✅ compareTo 작성 요령
- Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일 타임에 정해진다.
즉, 입력 인수의 타입을 확인하거나 형변환할 필요가 없다! (인수의 타입이 잘못됐다면 컴파일 자체가 되지 않는다.)
또한 null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.
- compareTo 메서드는 각 필드가 동치인지를 비교하는 게 아니라 그 **순서를 비교**한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면, `비교자(Comparator)`를 사용한다.
비교자는 직접 만들거나(비교자 생성 메서드) 자바가 제공하는 것 중에 골라 쓰면 된다.

#### 🔴 String 타입 예제 - 객체 참조 필드가 하나뿐인 비교자
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    private final String s;
    
    // 자바가 제공하는 비교자를 사용해 클래스를 비교한다.
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ...
}

```
`Comparable<CaseInsensitiveString>`
    - CaseInsensitiveString의 참조는 CaseInsensitiveString 참조만 비교할 수 있다는 의미이다.

#### 🔴 정수 기본 타입 예제 - 기본 타입 필드가 여럿일 때의 비교자
박싱된 기본 타입 클래스는 정적 메서드인 compare를 이용할 수 있다.
```java
public int compareTo(PhoneNumber pn) {
    //가장 중요한 필드
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0)  {
        //두번째로 중요한 필드
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            //세번째로 중요한 필드
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}

```

이렇게 클래스에 핵심 필드가 여러개 라면, compare를 통해 핵심 필드부터 똑같지 않은 필드를 찾을 때까지 비교해나간다.

### ✅ 비교자 생성 메서드 (comparator construction method)
자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
이 비교자들을 compareTo 메서드 구현하는 데 잘 활용할 수 있다.
이 방식은 간결하지만, 약간의 성능 저하가 뒤따른다.

#### 🔴 int 타입 등 숫자 타입용 비교자 생성 메서드
```java
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
            .thenComparingInt(pn -> pn.prefix)
            .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}

```
이 코드는 클래스를 초기화할 때 비교자 생성 메서드 2개를 이용해 비교자를 생성한다.

1️⃣ comparingInt()
- 키 추출 함수 (key extractor function)를 인수로 받아, 그 키(int 타입)를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드.
- (PhoneNumber pn) -> ... : 타입을 명시해야 한다.

2️⃣ thenComparingInt()
- int 키 추출자 함수를 입력받아 다시 비교자를 반환하는 Comparator의 인스턴스 메서드.
- 첫 번째 비교자를 적용한 다음 새로 추출한 키로 추가 비교를 수행한다. thenComparingInt는 원하는 만큼 연달아 호출할 수 있다.
- pn -> ... : 여기서는 타입을 명시하지 않았다. 자바의 타입 추론 능력이 이 정도는 추론해낼 수 있기 때문이다.

Comparator는 이외에도 수많은 보조 생성 메서드들이 존재한다.

`comparingLong`, `comparingDouble`, `thenComparingLong`, `thenComparingDouble`

이와 같이
long과 double 용으로 comparingInt와 thenComparingInt의 변형 메서드가 있다.
short처럼 더 작은 정수 타입에는 int용 버전을 사용하고, float도 마찬가지로 double용을 사용한다.
이런 식으로 자바의 숫자용 기본 타입을 모두 커버한다.

#### 🔴 객체 참조용 비교자 생성 메서드

`comparing`
comparing 정적 메서드는 2개가 다중정의되어 있다.

```java
public static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor)

public static <T, U> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor, Comparator<? super U> keyComparator)
```

- 첫 번째는, 키 추출자를 받아서 그 키의 자연석 순서를 사용한다.
- 두 번째는, 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받는다.

`thenComparing`
thenComparing 인스턴스 메서드는 3개가 다중정의되어 있다.

```java
default Comparator<T> thenComparing(Comparator<? super T> other)

default <U extends Comparable<? super U>> Comparator<T> thenComparing(Function<? super T, ? extends U> keyExtractor)
            
default <U> Comparator<T> thenComparing(Function<? super T, ? extends U> keyExtractor, Comparator<? super U> keyComparator)
```

- 첫 번째는, 비교자 하나만 받아서 그 비교자로 부차 순서를 정한다. (comparing으로 1차적 순서를 정하고, 이어지는 thenComparing으로 2차, 3차, .. 순서를 정한다.)
- 두 번째는, 키 추출자를 받아서 그 키의 자연석 순서로 보조 순서를 정한다.
- 세 번째는, 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받는다.


### 📌 핵심 정리
순서를 고려해야 하는 값 클래스를 작성해야 한다면 꼭 Comparable 인터페이스를 구현하자.
그 인스턴스들을 쉽게 정렬하고 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.
compareTo 메서드에서 필드의 값을 비교할 때 '<' 와 '>' 연산자는 쓰지 말아야 한다.
그 대신 박싱된 기본 타입 클래스가 제공하는 정적 `compare` 메서드나 Comparator 인터페이스가 제공하는 `비교자 생성 메서드`를 사용하자.

---

### 📌 Reference
- 이펙티브 자바
- [[Effective Java] 아이템14: comparable을 구현할지 고려하라](https://wisdom-cs.tistory.com/43)

