# String in Java

### **1. Java의 String은 불변(Immutable) 객체**

- `String` 객체는 한 번 생성되면 수정할 수 없음.
- 문자열을 변경하는 메서드를 호출하면 **새로운 객체**가 생성됨.

```java
public class Main {
    public static void main(String[] args) {
        String haribo1st = new String("HARIBO");
        String copiedHaribo1st = haribo1st.toUpperCase();
        String copiedHaribo1st1 = haribo1st.toLowerCase();

        System.out.println(haribo1st+" "+ copiedHaribo1st+ " "+copiedHaribo1st1 ); // HARIBO HARIBO haribo

        System.out.println(haribo1st == copiedHaribo1st); // true
        System.out.println(haribo1st == copiedHaribo1st1); // false
    }
}
```

- `toUpperCase()` 메서드는 내부적으로 변경이 필요 없으면 **기존 객체를 그대로 반환**함.
- `HARIBO`는 대문자로 변환할 필요가 없으므로 **같은 객체를 반환**하여 `true`가 나옴.

### **2. `new String()`을 사용한 경우**

- `new String("HARIBO")`는 **Heap 영역에 새로운 객체를 생성**.
- `"HARIBO"` 리터럴과 **다른 객체**임.

```java
public void func() {
    String haribo1st = new String("HARIBO");
    String haribo3rd = "HARIBO";

    System.out.println(haribo1st == haribo3rd);   // false (다른 객체)
    System.out.println(haribo1st.equals(haribo3rd)); // true (값은 같음)
}

```

- `==` → 객체 비교 (주소 비교)
- `equals()` → 값 비교

### **3. `String.valueOf()`의 경우**

- `String.valueOf("HARIBO")`는 내부적으로 `toString()`을 호출함.
- `String.toString()`은 `this`를 반환하므로 기존 객체를 그대로 사용.

```java
public void func() {
    String haribo3rd = "HARIBO";
    String haribo4th = String.valueOf("HARIBO");

    System.out.println(haribo3rd == haribo4th);   // true (같은 객체)
    System.out.println(haribo3rd.equals(haribo4th)); // true (값도 같음)
}

```

- `String.valueOf()`는 기존의 문자열 리터럴을 그대로 반환하므로 같은 객체.

### **4. 리터럴 문자열과 Constant Pool**

- JVM은 **Constant Pool**을 사용하여 **중복된 문자열 객체 생성을 방지**함.
    - 만약 `String str =  "abc";` 이렇게 선언한다면?
        - `"abc"`는 JVM의 String Constant Pool(문자열 상수 풀)에 저장됨
        - String Constant Pool은 **Heap 영역에 존재**하지만, 일반적인 Heap 객체와는 다르게 중복 저장을 방지함.
        - `str` 변수 자체는 스택(Stack) 메모리에 저장되고, `str`은 `"abc"` 객체의 **참조(주소)**를 가지고 있음
        - 메모리 구조
            
            ```java
            [Stack]      [Heap]
             str ──────► "abc" (Constant Pool)
            ```
            
- `new String("abc")`와 비교
    
    ```java
    String str1 = "abc";             // Constant Pool 사용
    String str2 = new String("abc"); // Heap에 새로운 객체 생성
    ```
    
    - `str1`은 `"abc"` 리터럴을 **Constant Pool에서 가져와** 참조.
    - `str2`는 **Heap에 새로운 객체를 생성**하고 `"abc"` 값을 복사.
    - `str1 == str2` → `false` (주소 다름)
    - `str1.equals(str2)` → `true` (값 같음)
    
    ```java
    [Stack]      [Heap]
     str1 ──────► "abc" (Constant Pool)
     str2 ──────► "abc" (Heap, new String 생성)
    
    ```
    

### **5. `String.intern()`의 역할**

- `intern()` 메서드는 **Heap에 있는 문자열을 Constant Pool에 등록**하여 동일한 문자열 리터럴이 있을 경우 해당 객체를 반환.

```java
java
복사편집
String s1 = new String("HARIBO");
String s2 = s1.intern(); // Constant Pool에 있는 "HARIBO"를 반환

System.out.println(s1 == s2); // false (s1은 Heap, s2는 Constant Pool)

```

- `intern()`을 사용하면 Heap에 있던 문자열을 Constant Pool로 옮길 수 있음.
- 성능 최적화를 위해 `intern()`을 적절히 활용 가능

## Heap & Constant Pool

```java
public class ConstantPoolExample {
    public static void main(String[] args) {
        String str1 = "hello";   // String Constant Pool에 저장
        String str2 = "hello";   // 기존 "hello" 객체 재사용
        String str3 = new String("hello"); // Heap에 새로운 객체 생성

        System.out.println(str1 == str2); // true (같은 객체 참조)
        System.out.println(str1 == str3); // false (Heap에 새로 생성됨)
    }
}

[Stack]         [Heap] (일반 객체)    [Heap] (Constant Pool)
 str1 ───┐                                   "hello"
 str2 ───┴──────────────────────────────┐
 str3 ──────► (new String("hello")) ───► X (새로운 객체)

```