# [JPA] Transaction

## JPA에서 Transaction의 기본 동작 방식

JPA에서 **트랜잭션(Transaction)은 데이터베이스의 일관성을 유지하기 위한 기본 단위이다.**

- JPA에서 데이터를 변경하려면 **반드시 트랜잭션이 필요**해.
- 스프링에서는 `@Transactional`을 사용하여 트랜잭션을 관리함.

### 📌 JPA에서 트랜잭션 동작 순서

1. **EntityManager가 영속성 컨텍스트를 생성**
2. **트랜잭션을 시작 (`begin()`)**
3. **엔티티 변경 감지 (Dirty Checking) → 변경된 엔티티를 자동 반영**
4. **트랜잭션 커밋 (`commit()`)** → `flush()` 실행 후 DB에 반영
5. **트랜잭션 종료 (`close()`)**

### ✔ JPA 트랜잭션 예제

```java
@Service
public class MemberService {
    @PersistenceContext
    private EntityManager em;

    @Transactional  // Spring에서 트랜잭션 관리
    public void updateMember(Long memberId, String newName) {
        Member member = em.find(Member.class, memberId);
        member.setName(newName);  // Dirty Checking 발생
    }
}
```

✅ **트랜잭션이 적용되어 `commit()`이 실행되면, 변경된 `member` 엔티티가 자동으로 업데이트됨.**

## **JPA에서 Transaction의 Isolation Level (격리 수준)**

트랜잭션의 **격리 수준(Isolation Level)은** 여러 트랜잭션이 동시에 실행될 때 데이터 정합성을 어떻게 유지할지 결정하는 기준이다.

JPA 자체적으로 격리 수준을 설정할 수는 없고, 데이터베이스의 격리  수준을 따름

📌 **트랜잭션 격리 수준 4가지**

| **격리 수준** | **설명** | **해결하는 문제** | **단점** |
| --- | --- | --- | --- |
| READ UNCOMMITTED | 다른 트랜잭션의 `commit` 되지 않은 데이터도 읽을 수 있 | Dirty Read | 데이터 정합성 낮음 |
| READ COMMITED
(기본값) | `commit` 된 데이터만 읽을 수 있음 | Dirty Read 해결 | Phantom Read 가능 |
| REPEATABLE READ | 한 트랜잭션 내에서 같은 데이터를 반복 조회하면 항상 같은 결과 반환 | Dirty Read, Non-Repeatable Read 해결 | Phantom Read 가능 |
| SERIALIZABLE | 트랜잭션을 순차적으로 실행 (가장 강력한 격리 수준) | 모든 문제 해결 | 성능 저하 |

### ✔ Spring에서 격리 수준 설정 방법

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void process() {
    // Repeatable Read 적용
}
```

## JPA에서 Transaction Propagation (전파 방식)

트랜잭션 전파(Propagation)는 **메서드가 실행될 때 기존 트랜잭션을 유지할지, 새로운 트랜잭션을 만들지 결정하는 옵션이다.**

### 📌 트랜잭션 전파 방식 종류

| **전파 속성** | **설명** |
| --- | --- |
| REQUIRED(기본값) | 기존 트랜잭션이 있으면 그대로 사용, 없으면 새로 생성 |
| REQUIRES_NEW | 기존 트랜잭션을 중단하고 새로운 트랜잭션을 생성 |
| SUPPORTS | 트랜잭션이 있으면 사용하고, 없으면 트랜잭션 없이 실행 |
| NOT_SUPPORTED | 트랜잭션 없이 실행 (기존 트랜잭션이 있어도 무시) |
| MANDATORY | 반드시 기존 트랜잭션이 있어야 함 (없으면 예외 발생) |
| NEVER | 트랜잭션이 없어야 실행됨 (있으면 예외 발생) |
| NESTED | 부모 트랜잭션 내에서 새로운 중첩 트랜잭션 생성 |

### ✔ Spring에서 트랜잭션 전파 설정 예제

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void processNewTransaction() {
    // 새로운 트랜잭션에서 실행
}
```

✅ **기존 트랜잭션이 있어도 새로운 트랜잭션을 만들어 실행함.**

## 트랜잭션과 Lazy Loading (지연 로딩)

- `@Transactional` 메서드 안에서는 **지연 로딩된 엔티티를 정상적으로 조회**할 수 있음.
- **트랜잭션이 끝나면 영속성 컨텍스트가 닫히므로, Lazy Loading을 하면 오류 발생!**

### **✔ 예제**

```java
@Transactional
public void process() {
    Member member = memberRepository.findById(1L).get();
    System.out.println(member.getOrders());  // Lazy Loading 정상 동작
}
```

✅ `@Transactional` 안에서는 Lazy Loading이 문제없음.

### ✔ 트랜잭션 바깥에서 Lazy Loading을 하면?

```java
public void process() {
    Member member = memberRepository.findById(1L).get();
    System.out.println(member.getOrders());  // LazyInitializationException 발생
}
```

❌ 트랜잭션이 종료되면 `LazyInitializationException`이 발생함.

**➡ 해결 방법:**

1. `@Transactional`을 추가해서 영속성 컨텍스트 유지
2. `fetch join` 또는 `EntityGraph` 사용해서 한 번에 조회

### ✔ 트랜잭션과 ReadOnly 옵션

- 읽기 전용 트랜잭션이면 `@Transactional(readOnly = true)`를 설정하면 최적화 가능.
- **변경 감지(Dirty Checking) 기능을 비활성화**하여 성능 향상.

```java
@Transactional(readOnly = true)
public Member findMember(Long id) {
    return memberRepository.findById(id).orElseThrow();
}
```

### ✔ Nested Transaction (중첩 트랜잭션)

- `Propagation.NESTED`를 사용하면 **부모 트랜잭션 내에서 독립적인 중첩 트랜잭션을 실행**할 수 있음.
- **부모 트랜잭션이 롤백되면, 중첩 트랜잭션도 롤백됨.**

```java
@Service
public class OrderService {

    @Transactional
    public void createOrder() {
        orderRepository.save(new Order());

        try {
            paymentService.processPayment();
        } catch (Exception e) {
            System.out.println("결제 실패, 하지만 주문은 유지됨");
        }
    }
}

@Service
public class PaymentService {

    @Transactional(propagation = Propagation.NESTED)
    public void processPayment() {
        throw new RuntimeException("결제 오류 발생!");
    }
}
```

✅ `processPayment()`에서 예외가 발생해도 `createOrder()`의 트랜잭션은 유지됨.