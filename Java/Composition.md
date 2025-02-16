# Composition

컴포지션(Composition)은 객체 지향 프로그래밍(OOP)에서 **객체를 구성하는 방식 중 하나**로, 객체가 다른 객체를 포함하는 관계를 의미한다.

즉, **"has-a" 관계**를 표현하는 방식이다.

## 컴포지션의 개념

컴포지션은 상속(Inheritance)과 대비되ㅣ는 개념이다.

상속은 부모-자식 관계(”is-a” 관계)를 만들어 기능을 확장하는 방식이고, **컴포지션**은 한 객체가 다른 객체를 **속성으로 포함**하여 **기능을 재사용**하는 방식이다.

```java
class Engine {
    void start() {
        System.out.println("Engine started!");
    }
}

class Car {
    private Engine engine; // Car has an Engine

    public Car() {
        this.engine = new Engine();
    }

    void startCar() {
        engine.start();
        System.out.println("Car is running!");
    }
}

public class Main {
    public static void main(String[] args) {
        Car myCar = new Car();
        myCar.startCar();
    }
}
```

- `Car` 클래스는 `Engine` 객체를 **속성(필드)** 으로 가지고 있다.
- 이렇게 객체를 포함하여 기능을 활용하는 것이 컴포지션이다.

## 컴포지션 vs 상속

| 비교 항목 | 컴포지션(Composition) | 상속(Inheritance) |
| --- | --- | --- |
| 관계 | **has-a 관계** | **is-a 관계** |
| 유연성 | ✅ 높은 유연성 (객체 조합 가능) | ❌ 부모 클래스에 강하게 결합 |
| 캡슐화 | ✅ 캡슐화 유지 (외부에서 내부 구조 변경 가능) | ❌ 부모 클래스의 변경이 자식 클래스에 영향 |
| 다중 상속 문제 | ✅ 없음 (여러 객체 포함 가능) | ❌ Java는 다중 상속 지원 안 함 |

📌 **상속보다는 컴포지션이 더 유연하고 유지보수가 용이하다!**

그래서 **"상속보다는 컴포지션을 선호하라(Prefer Composition over Inheritance)"** 라는 원칙이 있다.

## 컴포지션을 활용한 예제 (HAS-A 관계)

```java
// 무기 인터페이스
interface Weapon {
    void attack();
}

// 칼 (Weapon 인터페이스 구현)
class Sword implements Weapon {
    public void attack() {
        System.out.println("칼로 베기!");
    }
}

// 활 (Weapon 인터페이스 구현)
class Bow implements Weapon {
    public void attack() {
        System.out.println("화살 쏘기!");
    }
}

// 캐릭터 클래스 (무기를 포함)
class Character {
    private Weapon weapon; // Character has a Weapon

    public Character(Weapon weapon) {
        this.weapon = weapon;
    }

    public void changeWeapon(Weapon newWeapon) {
        this.weapon = newWeapon;
    }

    public void attack() {
        weapon.attack();
    }
}

public class Main {
    public static void main(String[] args) {
        Character warrior = new Character(new Sword()); // 초기 무기: 칼
        warrior.attack(); // "칼로 베기!"
        
        warrior.changeWeapon(new Bow()); // 무기 변경: 활
        warrior.attack(); // "화살 쏘기!"
    }
}

```

- 컴포지션을 활용하면 `Character` 클래스가 특정 무기에 의존하지 않고, 다양한 무기를 조합할 수 있다.

## 컴포지션과 의존성 주입(DI)

Spring 같은 프레임워크에서는 **컴포지션을 활용한 의존성 주입(Dependency Injection, DI)** 을 자주 사용한다.

```java
class Car {
    private Engine engine;

    // 생성자를 통한 의존성 주입
    public Car(Engine engine) {
        this.engine = engine;
    }

    void startCar() {
        engine.start();
        System.out.println("Car is running!");
    }
}

public class Main {
    public static void main(String[] args) {
        Engine myEngine = new Engine();
        Car myCar = new Car(myEngine); // Engine 객체를 주입
        myCar.startCar();
    }
}

```

- **`Car` 클래스가 `Engine`을 직접 생성하지 않고, 외부에서 주입받는 방식**이다.
- 이렇게 하면 객체의 재사용성이 높아지고, `Engine`을 쉽게 교체할 수 있다.