# [JPA] Inheritance

JPA에서 **Inheritance(상속)**은 객체지향 프로그래밍의 **상속 개념을 관계형 데이터베이스에서 구현하는 방법**이다.
즉, **부모 클래스를 상속받는 자식 클래스들이 있을 때, 이를 하나의 테이블 또는 여러 개의 테이블로 매핑하는 전략을 의미**한다.
JPA는 상속 관계를 테이블에 매핑할 수 있도록 `@Inheritance` 애노테이션을 제공하며, 여러 가지 전략을 선택할 수 있다.

## JPA에서 Inheritance 전략

JPA에서는 다음과 같은 3가지 주요 상속 매핑 전략을 제공한다.
각 전략은 `@Inheritance(strategy = InheritanceType.XXX)` 애노테이션을 사용하여 지정할 수 있다.

1. **Single Table 전략** (`InheritanceType.SINGLE_TABLE`)
2. **Joined Table 전략** (`InheritanceType.JOINED`)
3. **Table per Concrete Class 전략** (`InheritanceType.TABLE_PER_CLASS`)

## Single Table 전략

- **Single Table 전략**은 **모든 상속 관계의 엔티티를 하나의 테이블에 저장**하는 방식이다.
- **즉, 부모 클래스와 자식 클래스의 모든 속성을 하나의 테이블에서 관리한다**.
    - 이를 구분하기 위해 `@DiscriminatorColumn` 을 사용하여 각 행이 어떤 엔티티에 해당하는지 표시할 수 있다.

### 특징

- 하나의 테이블에서 모든 데이터를 관리하므로 **조회 성능이 좋다.**
- 테이블이 단순해지고, **조인이 필요 없어서** **쿼리가 단순해진다.**
- 그러나 자식 엔티티마다 속성이 다를 경우, **불필요한 NULL 값이 많이 발생할 수 있다.**

### 예제 코드

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) // Single Table 전략
@DiscriminatorColumn(name = "DTYPE") // 엔티티 구분 컬럼
public abstract class Vehicle {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
}

@Entity
@DiscriminatorValue("CAR")
public class Car extends Vehicle {
    private int maxSpeed;
}

@Entity
@DiscriminatorValue("BIKE")
public class Bike extends Vehicle {
    private boolean hasPedals;
}

```

**생성되는 테이블 예시**

| id | name | DTYPE | maxSpeed | hasPedals |
| --- | --- | --- | --- | --- |
| 1 | 소나타 | CAR | 200 | NULL |
| 2 | 자전거 | BIKE | NULL | TRUE |

## Joined Table

**Joined Table 전략**은 **부모 클래스와 자식 클래스별로 각각 테이블을 생성하고, 이를 조인하여 데이터를 조회하는 방식이다.**

### 특징

- 테이블 정규화가 이루어져 **데이터 중복을 방지**할 수 있다.
- 자식 테이블에서 부모 테이블을 참조하기 때문에 **조인 성능이 저하될 가능성이 있다.**
- 테이블이 **정규화되므로, NULL 값이 적고 저장 공간을 절약할 수 있다.**

### 예제 코드

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // Join Table 전략
public abstract class Vehicle {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
}

@Entity
public class Car extends Vehicle {
    private int maxSpeed;
}

@Entity
public class Bike extends Vehicle {
    private boolean hasPedals;
}

```

**생성되는 테이블 예시**

- Vehicle 테이블
    
    
    | id | name |
    | --- | --- |
    | 1 | 소나타 |
    | 2 | 자전거 |
- Car 테이블
    
    
    | id | maxSpeed |
    | --- | --- |
    | 1 | 200 |
    - 🚗 자동차(Car) 데이터를 조회하는 경우
        
        ```sql
        SELECT v.id, v.name, c.maxSpeed 
        FROM Vehicle v 
        JOIN Car c ON v.id = c.id 
        WHERE c.id = 1;
        ```
        
    - 결과
        
        
        | id | name | maxSpeed |
        | --- | --- | --- |
        | 1 | 소나타 | 200 |
- Bike 테이블
    
    
    | id | hasPedals |
    | --- | --- |
    | 2 | TRUE |
    - 🚲 자전거(Bike) 데이터를 조회하는 경우
        
        ```sql
        SELECT v.id, v.name, b.hasPedals 
        FROM Vehicle v 
        JOIN Bike b ON v.id = b.id 
        WHERE b.id = 2;
        
        ```
        
    - 결과
        
        
        | id | name | hasPedals |
        | --- | --- | --- |
        | 2 | 자전거 | TRUE |

자식 테이블은 부모 테이블의 **PK를 그대로 상속받아** FK(외래키)로 설정되며, 이를 통해 부모 테이블과 조인하여 데이터를 가져온다.

## Table per Concrete Class

**Table per Concrete Class 전략**은 **부모 클래스를 위한 테이블을 만들지 않고, 자식 클래스별로 개별적인 테이블을 생성하는 방식**이다.

즉, 각 자식 클래스가 **부모의 속성까지 포함한 개별 테이블을 가진다.**

### 특징

- 부모 테이블이 없으므로 **조인이 필요 없어 조회 성능이 빠르다.**
- 하지만 **각 자식 테이블에 부모 속성이 중복 저장되므로, 데이터 중복이 발생할 수 있다.**
- `@GeneratedValue` 사용이 어려운 경우가 많아, ID를 직접 관리해야 할 수도 있다.

### 예제 코드

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO) // 테이블별 ID 생성 필요
    private Long id;
    private String name;
}

@Entity
public class Car extends Vehicle {
    private int maxSpeed;
}

@Entity
public class Bike extends Vehicle {
    private boolean hasPedals;
}
```

**생성되는 테이블 예시**

- Car 테이블
    
    
    | id | name | maxSpeed |
    | --- | --- | --- |
    | 1 | 소나타 | 200 |
- Bike 테이블
    
    
    | id | name | hasPedals |
    | --- | --- | --- |
    | 2 | 자전 | TRUE |

**데이터 중복 발생 예시**

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    private String name;
}

@Entity
public class Car extends Vehicle {
    private int maxSpeed;
}

@Entity
public class Bike extends Vehicle {
    private boolean hasPedals;
}
```

**생성되는 테이블 예시**

🚗 **Car 테이블 (자동차 데이터 저장)**

| id | name | maxSpeed |
| --- | --- | --- |
| 1 | 소나타 | 200 |
| 2 | BMW | 250 |

🚲 **Bike 테이블 (자전거 데이터 저장)**

| id | name | hasPedals |
| --- | --- | --- |
| 1 | 자전거 | TRUE |
| 2 | MTB | TRUE |

✅ **문제점:**

- `Vehicle` 테이블이 없고, **모든 자식 테이블이 name 필드를 중복 저장**
- `Car`와 `Bike` 테이블의 **ID 값이 서로 중복될 수 있음** → `id=1`이 두 테이블에 존재할 수 있음

**데이터 중복 발생 이유**

1. **부모 테이블을 따로 만들지 않기 때문**
    - `Joined Table` 전략처럼 부모 테이블에서 공통 필드를 관리하지 않음
    - 대신 자식 테이블이 부모 필드를 개별적으로 저장
2. **각 테이블이 독립적으로 존재하기 때문**
    - 조회할 때 테이블을 조인할 필요는 없지만, 그 대신 공통 필드(`name`)가 **각 테이블에 중복 저장됨**
    - `Vehicle` 테이블이 없으므로 `Car`와 `Bike` 테이블이 서로 관계가 없음
3. **ID가 중복될 수 있음**
    - `@GeneratedValue(strategy = GenerationType.AUTO)`를 사용하면 테이블마다 별도로 ID를 생성하기 때문에 **각 테이블에서 ID가 중복될 가능성이 높음**

**데이터 중복 발생 시 문제점**

❌ **정규화 깨짐** → 동일한 `name` 필드가 여러 테이블에 반복 저장됨

❌ **ID 중복 문제** → 서로 다른 테이블에서 같은 `id` 를 가질 수 있음

❌ **데이터 일관성 문제** → `Vehicle` 데이터를 한 번에 조회하기 어렵고, 중복된 데이터를 일괄 수정하기 어려움

**Table per Concrete Class 전략을 사용할 때 고려할 점**

- **읽기 성능이 중요한 경우** 사용 가능
- **자식 엔티티가 서로 독립적인 경우** 유용 → 부모-자식 관계가 약한 경우 선택 가능
- 하지만 데이터 중복과 ID 중복 문제를 해결해야 함
    - ID 중복 방지를 위해 `UUID` 또는 `TABLE 전략`을 활용할 수도 있음
    - 가능하면 `Joined Table` 또는 `Single Table` 전략을 고려하는 것이 일반적