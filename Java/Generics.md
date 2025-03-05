# Generics

**제네릭(Generics)**은 **클래스나 메서드에서 사용할 데이터 타입을 컴파일 시점에 지정할 수 있도록 하는 기능**

Java에서 타입의 안정성을 높이고 **코드의 재사용성을 증가**시키기 위해 도입되었다.

## 왜 제네릭이 필요할까?

❌ **제네릭이 없던 시절 (JDK 1.4 이하)**

```java
import java.util.*;

public class NonGenericList {
    public static void main(String[] args) {
        List list = new ArrayList(); // Object 타입 저장
        list.add("Hello");
        list.add(123);  // 다른 타입도 추가 가능 (타입 안정성 문제)

        String str = (String) list.get(1);  // ClassCastException 발생 가능
        System.out.println(str);
    }
}
```

- `List` 는 기본적으로 `Object` 타입을 저장하므로, 데이터를 꺼낼 때 명시적인 타입 캐스팅을 해야 한다.
- `Integer`를 `String`으로 캐스팅하면 **런타임 오류 (ClassCastException)** 발생

## 제네릭을 사용한다면?

```java
import java.util.*;

public class GenericList {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(); // String만 저장 가능
        list.add("Hello");
        // list.add(123); // ❌ 컴파일 에러 발생!

        String str = list.get(0);  // 캐스팅 필요 없음
        System.out.println(str);
    }
}
```

- `List<String>` → **컴파일 시점에 타입 체크 가능**
- `String str = list.get(0);` → **명시적 캐스팅 불필요**

## **제네릭의 장점**

1. **타입 안정성(Type Safety) 보장**
    - 컴파일 단계에서 타입을 강제하므로, **런타임 에러(ClassCastException) 방지**
2. **캐스팅(Casting) 불필요**
    - `Object` → `String` 형변환이 필요 없어 **성능 향상**
3. **코드 재사용성 증가**
    - 다양한 타입에서 동일한 코드 사용 가능

## 제네릭 클래스 만들기

```java
class Box<T> {  // T는 타입 매개변수 (Type Parameter)
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}

public class GenericExample {
    public static void main(String[] args) {
        Box<String> stringBox = new Box<>(); 
        stringBox.setItem("Hello");
        System.out.println(stringBox.getItem()); // Hello

        Box<Integer> intBox = new Box<>(); 
        intBox.setItem(100);
        System.out.println(intBox.getItem()); // 100
    }
}

```

- `Box<T>` → `T`는 **임의의 타입**을 의미
- `Box<String>`, `Box<Integer>` 처럼 **다양한 타입으로 활용 가능**

## 제네릭 메서드

```java
class Util {
    public static <T> void printArray(T[] array) {
        for (T item : array) {
            System.out.println(item);
        }
    }
}

public class GenericMethodExample {
    public static void main(String[] args) {
        Integer[] intArray = {1, 2, 3};
        String[] strArray = {"A", "B", "C"};

        Util.printArray(intArray);
        Util.printArray(strArray);
    }
}
```

- `<T>` 선언 후 `T` 타입을 메서드에서 사용
- **다양한 타입의 배열을 하나의 메서드에서 처리 가능!**

## 제네릭의 제한 (제한된 타입 매개변수)

### ✅ **상위 클래스 제한 (extends)**

```java
class NumberBox<T extends Number> {  // Number 또는 하위 타입만 가능
    private T number;

    public NumberBox(T number) {
        this.number = number;
    }

    public double doubleValue() {
        return number.doubleValue();  // Number의 메서드 사용 가능
    }
}

public class BoundedTypeExample {
    public static void main(String[] args) {
        NumberBox<Integer> intBox = new NumberBox<>(10);
        NumberBox<Double> doubleBox = new NumberBox<>(3.14);

        // NumberBox<String> strBox = new NumberBox<>("Hello"); ❌ 컴파일 에러
    }
}
```

- `T extends Number` → `Number`의 하위 타입(`Integer`, `Double` 등)만 허용
- `T`가 `Number`를 상속받았으므로, `doubleValue()` 같은 `Number`의 메서드 사용 가능

## 와일드카드

### ✅ **제네릭 타입의 범위를 넓힐 때 사용**

```java
class Printer {
    public static void printList(List<?> list) {  // 모든 타입의 List 허용
        for (Object obj : list) {
            System.out.println(obj);
        }
    }
}

public class WildcardExample {
    public static void main(String[] args) {
        List<Integer> intList = List.of(1, 2, 3);
        List<String> strList = List.of("A", "B", "C");

        Printer.printList(intList);
        Printer.printList(strList);
    }
}

```

- `List<?>` → **모든 타입의 List를 받을 수 있음**
- `List<Integer>`, `List<String>` 다 가능

### ✅ **와일드카드의 변형**

| **와일드 카드** | **설명** |
| --- | --- |
| `<?>` | 모든 타입 허용 (`List<?>`) |
| `<? extends T>` | `T` 또는 그 하위 타입 허용 (`List<? extends Number>`) |
| `<? super T>` | `T` 또는 그 상위 타입 허용 (`List<? extends Number>`) |

### `<? extends T>` 와 `<? super T>` 는 각각 언제 사용하면 좋을까?

- `<? extends T>` (상한 제한 와일드카드)
    - `T` **또는 그 하위 타입만 허용**하는 경우 사용
    - **읽기 전용 (Read-only) 성격이 강함**
    - 예: `List<? extends Number>` → `List<Integer>`, `List<Double>` 가능

📌 언제 사용하면 좋을까?

- 매개변수로 받은 리스트에서 **값을 읽기만 할 때**
    - 예: 특정 숫자 리스트를 받아 평균을 계산하는 경우
        
        ```java
        public static double sum(List<? extends Number> list) {
            double sum = 0;
            for (Number num : list) {
                sum += num.doubleValue();
            }
            return sum;
        }
        
        ```
        

- `<? super T>` (하한 제한 와일드카드)
    - `T` **또는 그 상위 타입만 허용**하는 경우 사용
    - **쓰기 전용 (Write-only) 성격이 강함**
    - 요소를 **추가할 수 있지만**, **정확한 타입을 보장받기 어려움**
    - 예: `List<? super Integer>` → `List<Integer>`, `List<Number>`, `List<Object>` 가능

📌 언제 사용하면 좋을까?

- 매개변수로 받은 리스트에 **값을 추가할 때**
    - 예: `List<Number>`에 `Integer` 값을 추가하는 경우
        
        ```java
        public static void addNumbers(List<? super Integer> list) {
            list.add(10);
            list.add(20);
        }
        ```
        

## `Object`와 `?`(와일드카드)의 차이

둘 다 "모든 타입을 받을 수 있다"고 볼 수 있지만, **용도가 다르고 동작 방식에도 차이가 있다.**

| **구분** | List<Object> | List<?> |
| --- | --- | --- |
| 타입 제한 | `Object` 타입의 요소만 추가 가능 | 모든 타입을 담고 있는 리스트를 받을 수 있음 |
| 요소 추가 | 가능 (`add new Object())`) | 불가능 (`add(null))` 만 가능 |
| 요소 읽기 | 가능 (`Object` 타입으로 변환) | 가능 (`Object` 타입으로 변환) |
| 제네릭 리스트와의 호환 | `List<String>`을 받을 수 없음
 | `List<String>`, `List<Integer>` 등 모든 리스트를 받을 수 있음 |

### 📌 `List<?>` 는 왜 요소 추가가 불가능할까?

- `List<?>`는 **"어떤 타입인지 모르지만, 특정한 타입의 리스트다"** 라는 의미이다.
- 즉, `List<String>`, `List<Integer>`, `List<Double>` 등 어떤 리스트가 올지 모르기 때문에 안정성을 보장할 수 없어서 요소 추가가 제한되는 것

```java
List<?> list = new ArrayList<String>();
list.add("Hello");  // ❌ 컴파일 에러! (어떤 타입인지 몰라서 추가 불가능)
list.add(null);  // ✅ 유일하게 허용됨 (null은 모든 타입과 호환 가능)

// 하지만 요소를 읽는 것은 가능해
// 왜냐하면 어떤 리스트이든 결국 Object 타입으로 캐스팅될 수 있기 때문이야.
Object obj = list.get(0);  // ✅ 가능 (Object 타입으로 읽기)
```

- `?`는 구체적인 타입을 알 수 없기 때문에, **어떤 타입이 들어올지 컴파일러가 모름**.
- 만약 `List<String>`이 들어왔다고 가정하면, `list.add(123);` 같은 코드가 실행될 수도 있는데,`String` 리스트에는 `Integer`를 추가할 수 없기 때문에 **타입 안정성을 보장할 수 없음**.
- 따라서 **`List<?>`에서는 요소를 추가할 수 없고, `null`만 추가 가능하다.**

## 타입 소거(Type Erasure)란 무엇이고, 제네릭과 어떤 관계가 있을까?

### ✅ 타입 소거(Type Erasure)란?

- 자바 컴파일러가 **제네릭 타입 정보를 제거**하고 **Object 또는 제한된 타입으로 변환하는 과정**
    - **제네릭은 컴파일 타임에서만 존재**하며, **런타임에는 사라짐**
    - 즉, **제네릭 타입 정보는 컴파일 후 bytecode에서 확인할 수 없음**

- 타입 소거 전

```java
public class Box<T> {
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}
```

- 타입 소거 후 (컴파일 후 변환된 코드)

```java
public class Box {
    private Object item;  // T → Object 로 변환됨

    public void setItem(Object item) {
        this.item = item;
    }

    public Object getItem() {
        return item;
    }
}
```

**📌 타입 소거가 일어나는 이유**

1. **자바의 하위 호환성 유지** (JDK 1.4 이하의 코드와 호환)
2. **성능 최적화** (제네릭 타입을 런타임에서 유지하면 오버헤드 발생)

**📌 타입 소거로 인해 발생하는 문제**

1. **런타임에서 타입 정보를 알 수 없음**
    - `List<String>` 과 `List<Integer>` 는 런타임에 같은 타입으로 취급됨
    - `instanceof` 로 특정 제네릭 타입을 확인할 수 없음
        
        ```java
        List<String> list = new ArrayList<>();
        if (list instanceof List<String>) {  // ❌ 컴파일 에러
            System.out.println("String 리스트입니다.");
        }
        ```
        
2. 제네릭 배열 생성 불가
    
    ```java
    List<String>[] arr = new ArrayList<String>[10];  // ❌ 컴파일 에러
    ```
    
3. 타입 캐스팅 문제 발생 가능
    
    ```java
    Box rawBox = new Box(); // 경고 발생 (unchecked warning)
    rawBox.setItem(10);  
    String str = (String) rawBox.getItem();  // ❌ 런타임 ClassCastException 발생
    ```