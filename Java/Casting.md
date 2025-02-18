# Casting

## 캐스팅이란?

데이터 타입을 변환하는 것

Java에서는 두 가지 종류의 캐스팅이 존재한다.

1. 기본 타입(Primitive Type) 캐스팅
    - 정수 ↔ 실수 변환 등, 값 자체를 변환
    - **자동(묵시적) 변환**과 **강제(명시적) 변환**으로 나뉨
2. 참조 타입(Reference Type) 캐스팅
    - 부모 클래스 ↔ 자식 클래스 간의 형 변환
    - `instanceof` 를 사용하여 안전하게 변환할 수도 있음

## 기본 타입 캐스팅 (Primitive Type Casting)

### 자동(묵시적) 형 변환 (Widening Conversion)

- 작은 범위 → 큰 범위 변환 시 자동 변환됨.
- 데이터 손실이 없음.

```java
int num = 10;
double d = num;  // 자동 변환 (int → double)

System.out.println(d);  // 10.0
```

**자동 변환 가능 예시**

- `byte → short → int → long → float → double`
- `char → int` (유니코드 변환)

### 강제(명시적) 형 변환 (Narrowing Conversion)

- 큰 범위 → 작은 범위 변환 시 **데이터 손실 가능**
- 명시적으로 `(타입)`을 사용해야 함.

```java
double d = 9.7;
int num = (int) d;  // 강제 변환 (double → int)

System.out.println(num);  // 9 (소수점 버림)
```

**강제 변환 필요 예시**

- `double → float → long → int → short → byte`
- `int → char` (유니코드 범위 주의)

## 참조 타입 캐스팅 (Reference Type Casting)

객체 간의 형 변환은 **상속 관계에서만 가능**하며, **업캐스팅(Upcasting)과 다운캐스팅(Downcasting)** 으로 나뉜다.

### 업캐스팅(Upcasting) - 자동 변환

- **자식 클래스 → 부모 클래스** 변환
- 명시적인 캐스팅 없이 자동 변환됨.
- 부모 타입의 기능만 사용 가능.

```java
class Parent {
    void show() {
        System.out.println("Parent 클래스");
    }
}

class Child extends Parent {
    void display() {
        System.out.println("Child 클래스");
    }
}

public class CastingExample {
    public static void main(String[] args) {
        Parent p = new Child(); // 업캐스팅 (자동 변환)
        p.show();  // 가능 (부모의 메서드는 호출 가능)
        
        // p.display();  // 오류! (부모 타입에서는 자식의 메서드 접근 불가)
    }
}
```

### 다운캐스팅(Downcasting) - 명시적 변환 필요

- **부모 클래스 → 자식 클래스** 변환
- 반드시 **명시적 캐스팅** 필요
- 변환 전에 `instanceof`로 검사하는 것이 안전함

```java
public class CastingExample {
    public static void main(String[] args) {
        Parent p = new Child();  // 업캐스팅
        Child c = (Child) p;  // 다운캐스팅 (명시적 변환 필요)

        c.display(); // 가능 (자식의 메서드 사용 가능)
    }
}

```

❌ 잘못된 다운캐스팅 예시

```java
Parent p = new Parent();
Child c = (Child) p; // 오류 발생! (실제로 Child 객체가 아님)
```

✅ `instanceof`를 활용한 안전한 다운캐스팅

```java
if (p instanceof Child) {
    Child c = (Child) p;
    c.display();
} else {
    System.out.println("다운캐스팅 불가능");
}
```