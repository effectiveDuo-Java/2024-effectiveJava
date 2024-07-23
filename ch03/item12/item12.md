## toString을 항상 재정의하라

### toString 메서드
기본적으로 Object 클래스의 `toString()` 메서드는 해당 인스턴스에 대한 정보를 문자열로 반환한다. 이때 반환되는 문자열은 클래스 이름과 함께 **@**가 사용되며,
그 뒤로 16진수 해시 코드가 추가된다.
실제로 `toString()` 메서드 내부를 본다면 다음과 같이 구현되었다.

```java
public String toString(){
	//클래스 설계도, 클래스 이름 , @ - 위치, 해시코드(객체 주소)를 16진수로
	return getClass().getName()+"@"+Integer.toHexString(hashCode());
}

```
따라서 재정의를 하지 않는다면 `클래스 이름@16진수로 표현한 해시코드`가 출력된다.

```java
System.out.println(phoneNumber); //PhoneNumber@adbbd
```
하지만, 우리가 원하는 건 `707-867-5309` 같은 그 클래스의 필드 값이다. 따라서 toString은 다음과 같은 형태로 재정의 되어야한다.

```java
@Override public String toString() {
    return String.format("%03d-%03d-%04d",
            areaCode, prefix, lineNum);
}

```

### toString 생성시 주의할 점

#### ✅ toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.
하지만 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 **맨해튼 거주자 전화번호부(총 14878637개)** 와 같은 요약 정보를 담아야 한다.

#### ✅ 포맷을 명시하기로 했으면, 명시한 포맷에 맞는 문자열과 객체를 상호전환 할 수 있는 정적 팩토리나 생성자를 함께 제공하는 것이 좋다.
값 클래스의 경우에는 더욱 포맷 명시, 명시한 포맷에 맞는 객체 생성 코드를 작성해야 한다.

다음은 생성자로 명시한 포맷에 따라 phoneNumber 객체를 만든 코드이다.

```java
private PhoneNumber(String phoneNumberString) {
    String[] split = phoneNumberString.split("-");
    this.areaCode = Short.parseShort(split[0]);
    this.prefix   = Short.parseShort(split[1]);
    this.lineNum  = Short.parseShort(split[2]);
}
```

그리고 문자열을 phoneNumber 객체로 변환하는 정적팩토리 메서드

```java
public static PhoneNumber of(String phoneNumberString) {
	String[] split = phoneNumberString.split("-");
	PhoneNumber phoneNumber = new PhoneNumber(
			Short.parseShort(split[0]),
			Short.parseShort(split[1]),
			Short.parseShort(split[2]));
	return phoneNumber;
}

```
#### ✅ 포맷을 명시하든 아니든 의도는 명확히 밝히자.

```java
/*
	이 전화번호의 문자열 표현을 반환한다.
	이 문자열은 'xxx-yyy-zzz' 형태의 12글자로 구성된다.
	
	지역 코드, 프리픽스, 가입자 번호이다.
	각각의 대문자는 10진수 숫자 하나를 나타낸다.
	전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,

	앞에서부터 0으로 채워나간다. 가입자 번호가 123이라면 마지막 네 문자는 0123이 된다.

*/
@Override public String toString() {
    return String.format("%03d-%03d-%04d",
            areaCode, prefix, lineNum);
}

```


#### ✅ toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
phoneNumber클래스는 지역 코드, 프리픽스, 가입자 번호용 접근자를 각각 제공해야 한다. 따라서 이 각 필드를 따로 조회할 수 있는 API를 만들어두자는 뜻이다.
만약 만들어 두지 않는다면, 클라이언트 입장에서 toString 반환값을 직접 파싱해야된다는 점 주의하자. 그렇게 되면 성능이 나빠지고 필요하지도 않는 작업을 하게 된다.


### 자동생성?
앞서 다룬 equals,hashCode 처럼 toString도 AutoValue 프레임워크를 사용하면 자동 재정의가 가능하다.(대부분의 IDE도 마찬가지.)
다만 각 필드의 내용을 잘 나타내 주기도 하지만 클래스의 의미까지는 파악하지 못한다는 점 주의하자.

---

### 📌 Reference

- 이펙티브 자바
- [[이펙티브 자바] 12. toString을 항상 재정의하라](https://velog.io/@0sunset0/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-toString%EC%9D%84-%ED%95%AD%EC%83%81-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC)

- [자바 toString 오버라이딩 - 완벽 이해하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-toString-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%9E%AC%EC%A0%95%EC%9D%98-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

