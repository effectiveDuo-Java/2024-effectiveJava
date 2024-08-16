## 톱레벨 클래스는 한 파일에 하나만 담으라
소스 파일 하나에 여러 톱레벨 클래스를 선언하면 심각한 문제가 발생할 수 있습니다.  

- SH.java
```java
class SH {
    static final String NAME = "상혁";
}

class YH {
    static final String NAME = "영현";
}
```

- YH.java
```java
class SH {
    static final String NAME = "상혁";
}

class YH {
    static final String NAME = "영현";
}
```  

- Main.java
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(SH.NAME + YH.NAME);
    }
}
```  
위 코드를 실행하면 우리의 똑똑한 IDE인 **IntelliJ**가 컴파일 오류를 만듭니다.  
```
java: duplicate class: org.example.SH
java: duplicate class: org.example.YH
```  

하지만 `javac` 명령어로 컴파일 시 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라질 수 있습니다.  

### 해결 방법  
- 한 개의 파일에 한 개의 톱레벨 클래스만 둔다.
- 정적 멤버 클래스 방식을 사용한다.

```java
public class EffectiveDuo {
    public static class SH {
        ...
    }

    public static class YH {
        ...
    }
}
```  
위 예제와 같이 정적 멤버 클래스 방식으로 해결 가능합니다.  

---

### 📌 Reference
- 이펙티브 자바
