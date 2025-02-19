# String, StringBuilder, StringBuffer

Java에서 문자열을 다룰 때 사용하는 대표적인 클래스인 `String`, `StringBuilder`, `StringBuffer`에 대해 자세히 알아보자.

## String

- `String`은 **불변(Immutable) 객체**이다.
- 한 번 생성되면 변경할 수 없으며, 변경할 경우 새로운 객체가 생성된다.
- 문자열 리터럴을 사용할 경우 **String Pool**을 활용하여 메모리 효율성을 높일 수 있다.

### **사용 예시**

```java
String str1 = "Hello";   // String Pool에 저장됨
String str2 = new String("Hello"); // Heap 영역에 새로운 객체 생성

str1 = str1 + " World";  // 새로운 문자열 객체 생성
```

- `"Hello"`라는 문자열을 `str1`에 할당한 후 `" World"`를 추가하면, 기존 문자열이 변경되는 것이 아니라 새로운 문자열 객체가 생성됨.
- 기존 문자열을 수정하는 것이 아니라 새로운 객체를 만들기 때문에, **반복적인 문자열 변경이 필요할 경우 메모리 낭비가 발생**할 수 있음.

## StringBuilder

- `StringBuilder`는 **가변(Mutable) 객체**로, 기존 문자열을 변경할 수 있음.
- 문자열을 변경해도 새로운 객체를 생성하지 않고, **내부 버퍼를 수정**함.
- **싱글 스레드 환경**에서 성능이 좋음.

### 사용 예시

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // 기존 문자열을 수정함
System.out.println(sb.toString());  // "Hello World"
```

- `append()`를 사용하여 문자열을 추가하면 **기존 객체를 변경**하므로, `String`보다 메모리 효율이 좋음.

## StringBuffer

- `StringBuffer`는 `StringBuilder`와 동일하게 **가변(Mutable) 객체**이지만, **스레드 안전(Thread-safe)** 하다는 차이가 있음.
- 내부적으로 **synchronized 키워드**가 적용되어 여러 스레드에서 동시에 접근할 경우 안전함.
- 하지만 **싱글 스레드 환경에서는 `StringBuilder`보다 성능이 느릴 수 있음**.

### 사용 예시

```java
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World");  // 기존 문자열을 수정함
System.out.println(sbf.toString());  // "Hello World"
```

- `StringBuffer`는 멀티 스레드 환경에서 **데이터를 안전하게 처리해야 할 경우** 사용.

## 🎯 정리

- **문자열이 변하지 않는다면 `String`을 사용하자.**
- **문자열을 자주 수정해야 한다면 `StringBuilder`가 좋다.**
- **멀티 스레드 환경에서는 `StringBuffer`를 고려하자.**