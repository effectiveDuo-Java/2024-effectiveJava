## equals는 일반 규약을 지켜 재정의하라

### equals 메서드란?
equals는 모든 객체의 부모 클래스 Object의 메서드이다. 자식 클래스에서 재정의하지 않으면, 인스턴스가 오직 자기 자신과만 동일한 객체를 비교하여
동일하다면 true, 그렇지 않다면 false를 반환한다.
equals를 재정의할 때 감수할 리스크를 피하기 위해 아래 상황 중 하나라도 해당이 된다면 재정의하지 않는다. 

### 재정의 하지 않아도 되는 경우
#### 1) 각 인스턴스가 본질적으로 고유할 때
값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다. Thread가 대표적인 예, equals 메서드는 이러한 클래스에 맞게 구현되었다. 

#### 2) 인스턴스의 '논리적 동치성'을 검사할 일이 없을 때
java.util.regex.Pattern은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지 검사하는 방법도 있다. 하지만 설계자는 이 방식이 필요하지 않다고 판단할 수 있다. 이의 경우 기본 equals만으로도 해결이 된다.

#### 3) 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때
예를 들어 대부분의 set 구현체는 `AbstractSet`이 구현한 equals를 상속받아 쓰고, List 구현체는 `AbstractList`로부터, Map 구현체는 `AbstractMap`으로부터 상속받아 그대로 쓴다. 

#### 4) 클래스가 private이거나 package-private이고 equals메서드를 호출할 일이 없을 때

```
🐱상혁 : 그렇다면 언제 재정의를 해야할까?
🌱영현 : 객체 식별성(물리적인 동일성 확인)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 이를 비교하도록 재정의되지 않았을 때야. Integer와 String과 같은 값 클래스가 여기 해당해. 다만, 값 클래스라고 해도 값이 동일한 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스이거나 enum이라면 equals를 재정의하지 않아도돼.
```

### equals 명세
Object 명세엔 equals 메서드에 대한 규약이 작성되어 있다. 내용은 다음과 같다. 
equals 메서드는 동치관계를 구현하며, 다음을 만족한다.

#### 1) 반사성
`null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.`
한마디로, 객체는 자기 자신과 같아야 한다.

#### 2) 대칭성
`null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x) true다.`
두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다. 
예시로 대소문자 여부를 가리지 않는 다음 클래스를 작성했다고 가정하자.
```
public final class CaseInsensitiveString {
	private final String s;
	public CaseInsensitiveString(String s) {
		this.s = Objects.requireNonNull(s);
		}
	
	@Override public boolean equals(Object o) {
		if (o instanceof CaseInsensitiveString)
			return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
		if (o instanceof String) // One-way interoperability!
			return s.equalsIgnoreCase((String) o);
		return false;
	}
	
}
```
위 클래스는 일방향으로 흐르기 때문에, 대칭성에 위배된다.

```
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s) // true
s.equals(cis) // false
```

따라서 이런 결과가 나온다. 그래서 이 문제를 해결하기 위해 String과의 대칭성도 보장하겠다는 생각을 버려야한다.
결국, equals 메서드에서 String 타입과 비교하는 부분을 제거하여, `CaseInsensitiveString 객체 간의 비교만 허용`하도록 수정

```
@Override public boolean equals(Object o) {
	return o instanceof CaseInsensitiveString &&
		((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```
#### 3) 추이성
`null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true이다.`
첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같다는 뜻이다. 간단해 보이지만, 위배하기 쉽다. 만약에 자식 클래스에서 값을 추가하는 경우가 그러하다. 

```
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o) {
		if (!(o instanceof Point))
			return false;
		
		Point p = (Point)o;
		return p.x == x && p.y == y;
	}

} 
```
```
public class ColorPoint extends Point {
	private final Color color;
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}
	
}
```
이제 Color Point에 값 비교를 하는 equals를 오버라이드 해보자.

```
// Broken - violates transitivity!
@Override public boolean equals(Object o) {
	if (!(o instanceof Point))
		return false;

	// If o is a normal Point, do a color-blind comparison
	if (!(o instanceof ColorPoint))
		return o.equals(this);

	// o is a ColorPoint; do a full comparison
	return super.equals(o) && ((ColorPoint) o).color == color;
}

```
대칭성을 지키기 위해 ColorPoint의 equals메서드에서 Point인 경우는 Color 비교를 하지 않도록 한다.

```
ColorPoint x = new ColorPoint(1,2,RED);
Point y = new Point(1,2);
ColorPoint z = new ColorPoint(1,2,BLUE);

x.equals(y) // true
y.equals(z) // true
x.equals(z) // false
```

그러나, 이 경우 대칭성은 지켜지지만, 추이성은 지켜지지 않는다. 
따라서 이러한 어려움들 떄문에 하위 클래스에서 상위 클래스에 값을 추가하는 것은 어렵다. `상속 대신 컴포지션을 활용하라`는 방법을 통해 우회하는 방안이 있다.

```
// Adds a value component without violating the equals contract
public class ColorPoint {
	private final Point point;

    //컴포지션 적용 부분, ColorPoint클래스가 Point객체를 필드로 포함.
	private final Color color;

    //ColorPoint 안에 Point객체를 내부적으로 포함함으로써, 더 유연한 객체 구성 가능.
	public ColorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	//ColorPoint객체를 Point객체로 변환하여 반환. 이를 통해 ColorPoint 객체를 Point 객체처럼 사용 가능.
	public Point asPoint() {
	return point;
	}

	@Override public boolean equals(Object o) {
		if (!(o instanceof ColorPoint))
			return false;

		ColorPoint cp = (ColorPoint) o;
		return cp.point.equals(point) && cp.color.equals(color);
	}

}
```
따라서 이렇게 컴포지션을 활용하여 ColorPoint와 Point를 분리하면, equals 메서드에서 발생할 수 있는 추이성 문제를 해결할 수 있습니다.


#### 4) 일관성
일관성은 두 객체가 같다면 (둘 중 어느 하나도 수정되지 않았을 때) 앞으로도 영원히 같아야 한다는 뜻이다.
가변 객체는 상태가 바뀌면 equals 결과가 달라질 수 있지만, 불변 객체인 경우 그래선 안 된다.(상태가 안 변하는 것이 불변이기 때문에)
불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해선 안 된다. 
java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다. 호스트 이름으로 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 해당 결과가 항상 같다고 보장할 수 없기 때문에 종종 오류를 일으키는 원인이 된다. 이후 호환성이 발목을 잡아 영영 수정할 수 없는 내용이 되어버렸다. 이를 교훈삼아 항상 메모리에 존재하는 객체만을 사용한 계산을 수행하여야 한다.

#### 5) null이 아님
모든 객체가 null과 같지 않아야 한다는 의미이다. 보통 null check를 통해 할 수 도 있으나, instancof 연산자가 비교대상이 null이면 false를 반환하기 때문에 instance 연산자를 사용함으로써 묵시적 null check를 할 수 있다.  

```
@Override public boolean equals(Object o) {
	if (!(o instanceof MyType))
		return false;
	MyType mt = (MyType) o;
	...
}
```

### equals 제대로 구현하기
위의 규약을 준수하는 equals를 작성하려면 다음 step을 따라보자.
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. (성능 최적화를 위해)
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않다면 false를 반환한다.
3. 입력을 올바른 타입으로 형변환 한다. (instanceof를 통해 타입체크를 했기 때문에 오류가 발생하지 않는다.)
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
이 때 float과 double이 아닌 기본 필드는 ==을 통해 비교하고, 참조 타입은 equals로, float과 double은 Float.compare(), Double.compare() 메서드를 이용한다.
null 값을 정상적이라고 취급하는 객체라면, NPE를 방지하기 위해 Objects.equals(a,b); 메서드를 이용하자.

아래는 이 네가지를 준수한 예시이다.

```
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
    * Returns the point-view of this color point.
    */
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        // 1: == 연산자를 사용해 입력이 자기 자신의 참조인지 확인
        if (this == o) return true;

        // 2: instanceof 연산자로 입력이 올바른 타입인지 확인
        if (!(o instanceof ColorPoint)) return false;

        // 3: 입력을 올바른 타입으로 형변환
        ColorPoint cp = (ColorPoint) o;

        // 4: 핵심 필드들이 모두 일치하는지 검사
        return Objects.equals(point, cp.point) && Objects.equals(color, cp.color);
    }

    /..../
}

```

이렇게 equals를 구현했다면 앞서 말한것대로 대칭적인가? 추이성이 있는가? 일관성이 있나? 를 확인해보자.


### 이외에 구현시 주의사항

- equals를 재정의 할 때는 hashcode도 반드시 재정의하자.
- 또 너무 복잡하게 해결하려고 하지 말자. 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
- Object 외의 타입을 매개변수로 받는 equals메서드를 정의하지 말자. 

```
public boolean equals(MyClass o) {
    ...
}
```
위 메서드는 Object.equals를 재정의한 게 아니다. 입력 타입이 Object가 아니므로 재정의가 아니라 다중정의한 것이다. 

```
🐱상혁 : 이렇게 복잡한 절차를 다 겪어가며 equals 재정의를 해야하는걸까?
🌱영현 : 그래서 꼭 필요한 경우가 아니라면 재정의하지 않는 것이 좋아. 대부분 Object의 equals가 우리가 원하는 비교를 정확히 수행해줘.
        다만 재정의 해야한다면, 그 클래스의 핵심 필드 모두를 빠짐없이, 앞서 말한 다섯 가지 규약을 확실히 지켜가며 비교해줘야해!!
```
