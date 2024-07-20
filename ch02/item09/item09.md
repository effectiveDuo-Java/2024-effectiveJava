## try-finally보다는 try-with-resources를 사용하라  
자바에는 직접 close 메서드를 호출해서 닫아줘야 하는 자원이 많습니다.  

`InputStream`, `OutputStream`, `java.sql.Connection` 등이 해당합니다.  

자원 닫기를 제대로 하지 않으면 성능 문제로 이어지기도 합니다.  

앞서 [Item08 - finalizer와 cleaner 사용을 피하라](https://github.com/effectiveDuo-Java/2024-effectiveJava/blob/main/ch02/item08/item08.md) 에서 학습한 finalizer를 활용 가능하지만, finalizer은 단점이 너무나 많기에 믿음직스럽지 못합니다.  

### try-finally  
`try-finally` 구문은 예외가 발생하거나 메서드에서 반환되는 경우를 포함해서 전통적으로 자원을 닫기 위해 사용된 수단입니다.  

```java
public static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```  

만약 close를 여러번 호출할 상황이 오면 어떨까요?

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
	} finally {
		in.close();
	}
}
```  

위 두 코드 예제에는 미묘한 결점이 있습니다.  

예를 들어 기기에 물리적인 문제가 발생하면 `firstLineOfFile` 메서드의 `readLine` 메서드가 예외를 던지고, 같은 이유로 `close` 메서드도 실패할 것입니다.  

그럴 경우, 스택 추적 내역에는 첫 번째 예외 정보가 남지 않게되며 디버깅을 어렵게 만듭니다.  

이러한 문제들은 `try-with-resources`가 등장하며 해결되었습니다.  

### try-with-resources
Java 7에서 등장했으며 사용하기 위해선 자원이 `AutoCloseable` 인터페이스를 구현해야 합니다.  

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```  

이를 기반하여 앞의 문제가 있는 예제들을 수정해보겠습니다.  

```java
static String firstLineOfFile(String path) throws Exception {
	try (BufferedReader br = new BufferedReader (
		new FileReader(path))) {
			return br.readLine();
	}
}
```

```java
static void copy(String src, String det) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n == in.read(buf)) > = 0)
			out.write(buf, 0, n);
}
```  

### try-with-resources 장점  
수정된 코드를 통해 확인 가능하듯이 다음의 장점을 가지고 있습니다.  

- 짧고 읽기 수월하다  
- 문제를 진단하기에 용이하다  

`firstLineOfFile` 메서드를 살펴보면 `readLine`과 `close`에서 예외가 발생하면, `close`의 예외는 숨겨지고 `readLine` 예외만 기록됩니다.  

문제 있는 코드들과 달리 숨겨진 예외는 사라지지 않고 스택 추적 내역에 **suppressed** 꼬리표가 붙어 출력됩니다.  

Java 7에서 Throwable - getSuppressed 메서드를 사용하면 프로그램 코드에서 가져올 수 있습니다.  

또한, `try-with-resources` 구문에서도 `catch` 절을 사용할 수 있습니다.  

```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(
        new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        } 
}

```  

이로써, 파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환 가능 합니다.

---

### 📌 Reference

- 이펙티브 자바