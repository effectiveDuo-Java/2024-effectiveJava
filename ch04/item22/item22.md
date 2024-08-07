## 인터페이스는 타입을 정의하는 용도로만 사용하라
인터페이스는 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 얘기하는 용도로만 사용해야 합니다.  
하지만 항상 엇나가는 친구가 한둘은 꼭 있듯이 `상수 인터페이스(Constant interface)` 라는 친구가 있습니다.  


### 상수 인터페이스(Constant interface)
상수 인터페이스란 오직 상수만 정의한 인터페이스입니다.  
특징은 오직 static final 필드로 구성되어 있습니다.  

```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    double AVOCADOS_NUMBER = 6.022_140_857e23;
    
    // 볼츠만 상수 (J/K)
    double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량(kg)
    double ELECTRON_MASS = 9.109_383_56e-31;
}
```  
이는 인터페이스의 원래 목적과는 맞지 않는 인터페이스인데, 상수 인터페이스를 구현하는 것은 내부 구현을 클래스의 API로 노출하는 행위이기 때문입니다.  

```java
public class SH implements PhysicalConstants {
    // ...
}

public class YH implements SH {
    void effectiveMethod() {
        System.out.println(AVOCADOS_NUMBER); // YH에서 AVOCADOS_NUMBER 사용 가능
    }
}
```  
이렇게 될 경우 다음의 문제점들이 발생합니다.  
- 상수의 오염: 상위 클래스에서 정의된 상수들이 하위 클래스의 이름 공간에 포함되어 하위 클래스에서 의도하지 않게 사용될 수 있습니다.
- 네임스페이스 충돌: 하위 클래스가 상위 클래스에서 정의한 상수들과 동일한 이름의 상수를 정의하려고 할 때 충돌이 발생할 수 있습니다.
- 바이너리 호환성: 더 이상 상수들을 사용하지 않는다고 하더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현해야 합니다. 
    - 바이너리 호환성?
        - 프로그램 바이너리(컴파일 된 실행 파일)와 관련된 호환성 문제를 다룹니다.
        - 내부 구현의 변경이 클라이언트 코드에 영향을 미치지 않도록 하는 것을 말합니다. 
        - 즉, 클라이언트 코드가 상수의 값이 바뀌더라도 정상적으로 동작할 수 있도록 보장하는 것입니다.

<br></br>

위 상수 클래스의 문제점을 해결하기 위해 `final 클래스`을 사용하면 됩니다.  

```java
public final class PhysicalConstants {
    // 아보가드로 수 (1/몰)
    double AVOCADOS_NUMBER = 6.022_140_857e23;
    
    // 볼츠만 상수 (J/K)
    double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량(kg)
    double ELECTRON_MASS = 9.109_383_56e-31;
}
```  
이를 통해, 상속이 불가능하게 되어 상수들이 하위 클래스의 이름 공간에 오염되는 것을 방지합니다.  

```
🐱영현 : 그럼 상수를 공개하고 싶으면 어떻게 해?
🌱상혁 : 열거 방식과 유틸리티 클래스가 있어
```  

### 1. 열거 타입(Enum Type)

```java
public enum PhysicalConstantsEnum {
    AVOCADOS_NUMBER(6.022_140_857e23),
    BOLTZMANN_CONSTANT(1.380_648_52e-23),
    ELECTRON_MASS(9.109_383_56e-31);

    private final double value;

    PhysicalConstantsEnum(double value) {
        this.value = value;
    }

    public double getValue() {
        return value;
    }
}
```
### 2. 유틸리티 클래스(Utility Class)

```java
package effectivejava.chpater4.item22.constantutilityclass;

public class PhysicalConstantsUtil {
    private PhysicalConstantsUtil() {} // 인스턴스화 방지
    
    // 아보가드로 수 (1/몰)
    public static double AVOCADOS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량(kg)
    public static double ELECTRON_MASS = 9.109_383_56e-31;
}
```  

만약 위 유틸리티 상수 값들을 자주 사용한다 시으면 **정적 임포트(static import)**를 사용하여 클래스 이름을 생략 가능합니다.  

```java
import effectivejava.chpater4.item22.constantutilityclass.PhysicalConstants.*;

public class Test {
    double atoms(double mols) {
        return AVOCADOS_NUMBER * mols;
    }
    // ...
}
```

---

### 📌 Reference
- 이펙티브 자바
