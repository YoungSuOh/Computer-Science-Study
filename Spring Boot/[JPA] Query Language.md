# [JPA] Query Language

JPA에서 쿼리를 실행하기 위해 사용하는 언어는 **JPQL, 과 Criteria API 등이 있다.**

JPA는 기본적으로 객체 지향적인 접근 방식을 제공하므로, SQL이 아닌 **엔티티 객체를 대상으로 하는 쿼리 언어가 필요**하다.

### **✔ JPA에서 사용되는 주요 쿼리 언어**

1. **JPQL (Java Persistence Query Language)** → SQL과 비슷하지만 **테이블이 아니라 엔티티 객체를 대상으로 함**
2. **Criteria API** → 동적 쿼리를 객체 지향적으로 작성할 수 있도록 도와줌
3. **Native Query** → 기존의 SQL을 직접 실행할 수 있도록 해줌
4. **Named Query** → 미리 정의해두고 재사용할 수 있는 쿼리

## JPQL (JPA Query Language)란?

 JPQL은 JPA에서 제공하는 객체 지향적인 쿼리 언어이다.

- SQL과 비슷하지만, **테이블이 아니라 엔티티 객체(Entity)를 대상으로 쿼리를 작성**하는 것이 특징이야.
- SQL은 **데이터베이스 테이블 컬럼**을 대상으로 하지만, **JPQL은 엔티티와 엔티티 필드를 대상으로 함**
- JPA 구현체(Hibernate)는 **JPQL을 SQL로 변환**해서 실행해 줌

### ✔ JPQL 예제

```java
// SQL: SELECT * FROM Member WHERE age > 20;
String jpql = "SELECT m FROM Member m WHERE m.age > 20";
List<Member> members = entityManager.createQuery(jpql, Member.class)
                                    .getResultList();
```

✅ **JPQL의 특징**

- `SELECT m FROM Member m` → **테이블(Member)이 아니라 엔티티(Member)를 대상으로 조회**
- `m.age > 20` → **컬럼이 아니라 엔티티 필드를 기준으로 조건을 설정**

📌 **JPQL을 실행하면 실제 SQL로 변환되어 실행됨**

```sql
SELECT m.id, m.name, m.age FROM member_table m WHERE m.age > 20;
```

## Criteria API란?

Criteria API는 동적 쿼리를 객체 지향적으로 작성할 수 있도록 해주는 JPA의 기능이다.

- **JPQL은 문자열을 기반으로 하기 때문에, 동적 쿼리를 작성하기 어렵다는 단점이 있음.**
- 이를 해결하기 위해 **Criteria API는 코드 기반으로 쿼리를 작성할 수 있도록 도와줌.**
- **타입 세이프티(Type Safety)를 보장**하므로, 컴파일 시점에 문법 오류를 잡을 수 있음.

### ✔ Criteria API를 활용한 동적 쿼리 작성 예제

```java
// 엔티티 매니저 가져오기
CriteriaBuilder cb = entityManager.getCriteriaBuilder();

// CriteriaQuery 생성
CriteriaQuery<Member> query = cb.createQuery(Member.class);
Root<Member> m = query.from(Member.class);

// 조건 추가: age가 30보다 큰 회원 조회
query.select(m).where(cb.gt(m.get("age"), 30));

// 쿼리 실행
List<Member> resultList = entityManager.createQuery(query).getResultList();
```

✅ **JPQL을 동적으로 만들지 않아도 되므로 코드가 더 안전해지고 유지보수성이 높아짐.**

## Native Query란?

Native Query는 JPA에서 직접 SQL을 실행할 수 있도록 지원하는 기능

- JPQL이나 Criteria API를 사용하지 않고, **기존의 SQL을 그대로 사용할 수 있음.**
- 데이터베이스의 **고유한 SQL 문법(예: 특정 함수, JOIN, 인덱스 힌트 등)을 사용할 때 유용**
- 단, **JPA의 객체 지향적인 이점을 활용할 수 없고, 데이터베이스에 종속적이라는 단점이 있음.**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Query(value = "SELECT * FROM member WHERE age > ?1", nativeQuery = true)
    List<Member> findMembersByAge(int age);
}
```

- SQL을 직접 사용하므로, 데이터베이스의 고유 기능을 활용할 수 있다.

## Named Query란?

Named Query는 **미리 정의된 JPQL을 저장해두고 필요할 때 재사용할 수 있는 기능**이다.

- `@NamedQuery` 또는 `META-INF/orm.xml` 에 등록하여 사용 가능
- **JPQL을 재사용할 수 있어서 성능 최적화에 유리**함
- **쿼리 수정이 필요할 때 한 곳만 변경하면 되므로 유지보수성이 향상됨**

### `@NamedQuery` 사용 예제

```java
@Entity
@NamedQuery(
    name = "Member.findByName",
    query = "SELECT m FROM Member m WHERE m.name = :name"
)
public class Member {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private int age;
}
```

📌 Named Query 실행

```java
List<Member> members = entityManager.createNamedQuery("Member.findByName", Member.class)
                                    .setParameter("name", "John")
                                    .getResultList();
```

✅ **JPQL을 따로 작성할 필요 없이, `Member.findByName` 을 실행하면 자동으로 JPQL이 적용됨**