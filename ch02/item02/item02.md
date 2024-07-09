## 생성자에 매개변수가 많다면 빌더를 고려하라
```
🐱영현 : item01에서 배운 정적 팩터리는 좋긴 한데 선택적 매개변수가 많아지면 어떻게 해..?
🌱상혁 : 흠...
```
정적 팩터리와 생성자에는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 문제가 있습니다.

### 🪜 점층적 생성자 패턴(telescoping constructor pattern)
```
public class NutritionFacts {
	private final int servingSize;  // 필수
	private final int servings;     // 필수
	private final int calories;     // 선택
	private final int fat;          // 선택
	private final int sodium;       // 선택
	private final int carbohydrate; // 선택

	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, 0);
	}

		public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```

```
// 지방(fat)에 0을 작성한 것 처럼 설정하길 원치 않는 매개변수까지 값을 지정해줘야합니다.
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
### ✔️ 점층적 생성자 패턴의 단점
#### 매개변수 개수가 늘어나면 클라이언트 코드를 작성하거나 읽기 어렵다
매개변수가 몇 개인지 또 각 값의 의미가 무엇인지 하나하나 파악해야한다는 단점이 있습니다.

### 🫘 자바빈즈 패턴(JavaBeans pattern)
매개변수가 없는 생성자로 객체를 만들고 setter 메서드들을 사용해 매개변수의 값을 설정하는 방식입니다.

```
public class NutritionFacts {
	private int servingSize = -1;  // 필수
	private int servings = -1;     // 필수
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFacts() {}

	public void setServingSize(int val) { servingSize = val; }
	public void setServings(int val) { servings = val; }
	public void setCalories(int val) { calories = val; }
	public void setFat(int val) { fat = val; }
	public void setSodium(int val) { sodium = val; }
	public void setCarbohydrate(int val) {carbohydrate = val; }
}
```

```
// 점층적 생성자 패턴에 비해 확장하기 쉽고, 인스턴스를 만들기 쉽고, 가독성이 좋습니다.
NutritinFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

### ✔️ 자바빈즈 패턴의 단점
객체 하나를 생성하려면 여러개의 setter 메서드들을 호출해야하며 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓여집니다.

```
🐱영현 : 그럼 매개변수가 늘어나면 이에 대응가능한 좋은 방법은 없는거야?
🌱상혁 : 빌더 패턴이 있지~
```

### 👷 빌더 패턴(Builder pattern)
필수 매개변수들만으로 생성자를 호출해 빌더 객체를 만들 수 있습니다.
그런 다음 setter 메서드들로 원하는 매개변수들을 선택 가능합니다.

```
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
		private final int servingSize;  // 필수
		private final int servings;     // 필수
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this,servingSize = serginsSize;
			this.servings = servings;
		}

		public Builder fat(int val) {
			fat = val;
			return this;
		}

		public Builder sodium(int val) {
			sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	private NutirionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.fat;
		carbohydrate = builder.carbohydrate;
	}
}
```
```
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
	.calories(100)
	.sodium(35)
	.carbohydrate(27)
	.build();
```
점층적 생성자 패턴과 자바빈즈 패턴과는 달리 **자기 자신**을 반환하는 메서드들로 원하는 매개변수만을 선택해서 객체를 구성 가능합니다.

### 🍡 빌더 패턴 - 계층적 빌더
빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋습니다.

```
public abstract class Pizza {
	public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;

	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}

		abstarct Pizza build();

		protected abstract T self(); // 하위 클래스에서 이 메서드를 오버라이드 해서 this를 반환
	}

	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}
}
```

```
public class NyPizza extends Pizza {
	public enum Size { SMALL, MEDIUM, LARGE }
	private final Size size;

	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;

		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}

		@Override
		public NyPizza build() {
			return new NyPizza(this);
		}

		@Override
		protected Builder self() {
			return this;		
		}

		public NyPizza(Builder builder) {
			super(builder);
			size = builder.size;
		}
	}
}
```
- 크기(size) 매개변수를 필수로 받는 NyPizza  

```
public class Calzone extends Pizza {
	private final boolean sauceInside;

	public static class Builder extends Pizza.Builder<Builder> {
		private boolean sauceInsice = false;

		public Builder sauceInside() {
			sauceInsdie = true;
			return this;
		}
		
		@Override
		public Calzone build() {
			return new Calzone(this);
		}

		@Override
		protected Builder self() {
			return this;
		}

		private Calzone(Builder builder) {
			super(builder);
			sauceInside = builder.sauceInside;
		}
	}
}
```
- 소스 선택이 가능한 Calzone  

```
NyPizza pizza = new NyPizza.Builder(SMALL)
	.addTopping(SAUSAGE)
	.addTopping(ONION)
	.build();

Calzone pizza = new Calzone.Builder()
	.addTopping(HAM)
	.sauceInside()
	.build();
```
위 예제의 addTopping은 상속의 특징에 의해 사용가능하며 `return self()` 메서드에 의해 하위 타입이 반환됩니다.

### 👣 빌더 패턴 - 사용
매개변수가 4개 이상은 되어야 값어치를 하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있음을 명심하고 사용하면 됩니다!

---

### 📌 Reference

- 이펙티브 자바