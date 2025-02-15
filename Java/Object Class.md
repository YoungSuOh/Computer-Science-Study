# Object Class

모든 Java 클래스는 자동으로  `Object` 클래스를 상속받음

즉, `Object` 클래스는 Java에서 모든 클래스의 부모 역할을 한다.

## 주요 메서드

### `toString()`

- 객체의 문자열 표현을 반환하는 메서드
- 기본적으로 `"클래스명@16진수 해시코드"` 형태의 문자열을 반환한다.

```java
class Person {
    String name;
    Person(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "Person[name=" + name + "]";
    }
}

public class Main {
    public static void main(String[] args) {
        Person p = new Person("Alice");
        System.out.println(p);  // toString Overrride로 구현 안했을 경우 Person@3b07d329
        System.out.println(p.toString()); // Person[name=Alice]
        System.out.println(p); // Person[name=Alice]
        -> println() 메서드 내부에서 toString()을 호출
    }
}
```

### `hashCode()`

- 객체의 **해시 코드(고유한 정수 값)를 반환**하는 메서드
- 기본적으로 객체의 **메모리 주소 기반 해시 코드**를 반환
- `equals()`와 함께 오버라이딩하여 객체 비교에 활용

### **`equals(Object obj)`**

- 두 객체가 같은지를 비교하는 메서드
- 기본 구현은 **메모리 주소를 비교**하지만, `equals()`를 오버라이딩하면 객체의 필드 값을 비교 가능

```java
class Person {
    String name;
    Person(String name) { this.name = name; }
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj instanceof Person) {
            Person other = (Person) obj;
            return this.name.equals(other.name);
        }
        return false;
    }
}

```

### `wait()`, `notify()`, `notifyAll()`

- 스레드 간의 **동기화 및 통신을 담당하는 메서드**
- `synchronized` 블록 안에서만 사용 가능

```java
class SharedResource {
    synchronized void waitExample() throws InterruptedException {
        System.out.println("Thread is waiting...");
        wait();  // 락을 해제하고 대기
        System.out.println("Thread resumed.");
    }

    synchronized void notifyExample() {
        notify();  // 대기 중인 스레드 하나를 깨움
    }
    synchronized void notifyAllExample() {
        notifyAll();  // 대기 중인 스레드 모 깨움
    }
}

```

## **그 외 `Object` 메서드**

- **`clone()`**
    - 객체의 복제를 수행하는 메서드
    - `Cloneable` 인터페이스를 구현해야 사용 가능
- **예제:**
    
    ```java
    java
    복사편집
    class Person implements Cloneable {
        String name;
        Person(String name) { this.name = name; }
        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }
    ```
    

- **`finalize()`**
    - 객체가 **GC(Garbage Collector)에 의해 제거되기 직전에 호출되는 메서드**
    - 일반적으로 잘 사용되지 않으며, `try-with-resources`를 권장
- **예제:**
    
    ```java
    class Person {
        @Override
        protected void finalize() throws Throwable {
            System.out.println("객체가 제거됨.");
        }
    }
    
    ```
    

- **`getClass()`**
- 객체의 클래스 정보를 반환하는 메서드
- **예제:**
    
    ```java
    java
    복사편집
    Person p = new Person("Alice");
    System.out.println(p.getClass().getName()); // "Person"
    
    ```