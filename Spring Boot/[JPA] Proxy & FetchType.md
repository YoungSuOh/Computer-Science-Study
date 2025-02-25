# [JPA] Proxy & FetchType

## FetchType

`FetchType`은 **JPA에서 연관된 엔티티를 조회하는 방식(즉시 로딩 vs 지연 로딩)** 을 결정하는 설정이다.

연관 관계(`@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`)에서 `fetch` 속성을 설정하면, 쿼리 실행 시 즉시 로딩(`Eager`) 또는 지연 로딩(`LAZY`) 중 어떤 방식을 사용할지 결정할 수 있다.

### **1️⃣ `FetchType.EAGER` (즉시 로딩)**

```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "team_id")
private Team team;
```

✅ **설명:**

- **연관된 엔티티를 즉시 조회**하는 방식이야.
- 엔티티를 조회하면 연관된 엔티티도 **즉시 함께 조회됨(Join 쿼리 실행).**
- **기본값:** `@ManyToOne`, `@OneToOne`

✅ **쿼리 예시 (`EAGER` 로딩 시)**

```sql
SELECT * FROM member m
JOIN team t ON m.team_id = t.id
WHERE m.id = 1;
```

👉 `Member` 엔티티를 조회할 때 **`Team` 엔티티도 즉시 조회!**

✅ **장점:**

✔ 쿼리가 한 번만 실행되므로, **N+1 문제를 방지할 수 있음.**

✔ 자주 사용하는 관계라면 성능 최적화에 유리함.

✅ **단점:**

❌ **불필요한 데이터 로딩 발생 가능.**

❌ **모든 연관 엔티티를 미리 가져오므로 메모리 낭비 가능.**

### 2️⃣ `FetchType.LAZY` (지연 로딩)

```java
@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
private List<Member> members
```

✅ **설명:**

- **연관된 엔티티를 실제로 사용할 때(`get()` 호출 시) 조회**하는 방식이야.
- **기본값:** `@OneToMany`, `@ManyToMany`
- 처음에는 프록시 객체만 생성되고, 실제 데이터를 가져오지 않음.
- `getter` 호출 시 DB에서 데이터를 가져오는 쿼리가 실행됨.

✅ **쿼리 예시 (`LAZY` 로딩 시)**

```sql
SELECT * FROM member WHERE id = 1;  -- (초기 조회 시)
-- 이후 team.getName() 호출 시
SELECT * FROM team WHERE id = ?;
```

👉 `Member` 조회 시 `Team` 정보는 가져오지 않음.

👉 `team.getName()`을 호출하는 순간 `SELECT * FROM team` 실행됨!

✅ **장점:**

✔ **필요할 때만 데이터를 가져오므로 메모리 사용량이 줄어듦.**

✔ **연관된 엔티티를 사용할 일이 없으면 불필요한 쿼리가 실행되지 않음.**

✅ **단점:**

❌ **N+1 문제가 발생할 가능성이 있음.**

❌ **연관 객체를 사용할 때마다 추가 쿼리가 발생하여 성능 저하 가능.**

### 3️⃣ N+1 문제 (LAZY 로딩의 단점)

JPA에서 **N+1 문제** 란 **하나의 쿼리(1번)로 데이터를 조회했는데, 연관된 데이터를 가져오기 위해 추가적인 N개의 쿼리가 실행되는 문제** 를 말한다.

✅ **기본적인 엔티티 설정**

```java
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private List<Member> members = new ArrayList<>();
}

@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;
}
```

- `Team` 엔티티는 `@OneToMany` 관계로 `Member` 엔티티를 가지고 있다.
- `FetchType.LAZY` 설정 때문에 `members` 컬렉션을 처음 조회할 때는 DB에서 데이터를 가져오지 않는다.

✅ **예제 (`FetchType.LAZY` 사용 시 발생하는 문제) → 계속 Member table DB를 쳐야함**

```java
public List<Team> findAllTeams() {
    List<Team> teams = entityManager.createQuery("SELECT t FROM Team t", Team.class)
                                    .getResultList();

    for (Team team : teams) {
        System.out.println("팀 이름: " + team.getName());
        System.out.println("팀원 수: " + team.getMembers().size());  // 여기서 N+1 문제 발생!
    }
    return teams;
}
```

- `createQuery("SELECT t FROM Team t")` → `Team` 엔티티 목록을 가져오는 **1개의 쿼리 실행** (1번)
- `team.getMembers().size()` 를 호출할 때마다 **각 팀의 멤버를 가져오는 추가 쿼리 실행** (N번)

✅ **실행되는 SQL**

```java
SELECT * FROM Team;  -- (1) Team을 가져오는 쿼리 (1번 실행)

SELECT * FROM Member WHERE team_id = 1;  -- (N) 각 팀의 멤버를 가져오는 쿼리 (N번 실행)
SELECT * FROM Member WHERE team_id = 2;
SELECT * FROM Member WHERE team_id = 3;
...
```

- `Team`을 조회하는 쿼리는 1번 실행되지만, **각 Team의 members를 조회할 때마다 추가 쿼리가 실행됨**
- `팀 개수 = N` 이라면 `1 + N` 번의 SQL 실행 → **성능 저하 발생**

### 🎯 **최적화하는 방법**

1. `JOIN FETCH` 사용 (JPQL 기반)

- `JOIN FETCH` 를 사용하면 한 번의 SQL 쿼리로 연관된 엔티티까지 로딩 가능
    
    ```java
    public List<Team> findAllTeams() {
        return entityManager.createQuery("SELECT t FROM Team t JOIN FETCH t.members", Team.class)
                            .getResultList();
    }
    
    ```
    
- 실행되는 SQL (최적화됨)
    
    ```java
    SELECT t.*, m.* FROM Team t
    JOIN Member m ON t.id = m.team_id;
    ```
    
    - **장점**: 한 번의 쿼리로 `Team` + `Member` 데이터를 모두 가져올 수 있음
    - **단점**: 여러 개의 `JOIN FETCH` 는 사용할 수 없고, 여러 관계가 복잡하면 데이터 중복 발생 가능
1. `@EntityGraph` 사용 (Spring Data JPA)
    - `@EntityGraph` 를 사용하면 `JOIN FETCH` 와 비슷한 효과를 낼 수 있음
        
        ```java
        @EntityGraph(attributePaths = "members")
        @Query("SELECT t FROM Team t")
        List<Team> findAllWithMembers();
        ```
        
    - 실행되는 SQL
        
        ```java
        SELECT t.*, m.* FROM Team t
        LEFT JOIN Member m ON t.id = m.team_id;
        ```
        
        - **장점**: 기존 코드를 크게 수정하지 않아도 되고, 가독성이 좋음
        - **단점**: 복잡한 쿼리에는 적합하지 않음
2. `@BatchSize` 사용 (하이버네이트 기능)
    - `@BatchSize` 를 설정하면 한 번에 여러 개의 데이터를 조회하여 쿼리 개수를 줄일 수 있음
        
        ```java
        @Entity
        @BatchSize(size = 10)  // 한 번에 10개씩 가져오도록 설정
        public class Team {
            ...
        }
        ```
        
    - 실행되는 SQL
        
        ```sql
        SELECT * FROM Team;
        SELECT * FROM Member WHERE team_id IN (1, 2, 3, 4, ..., 10);
        ```
        
        ✅ 한 번에 여러 개의 팀 데이터를 가져오는 방식으로 **쿼리 개수를 줄일 수 있음!**
        

## JPA에서 Proxy란?

JPA에서 프록시(Proxy)란 지연 로딩(`FetchType.LAZY`)을 사용할 때, 실제 엔티티 대신 가짜 객체를 생성하여 사용하는 것을 의미한다.
이 프록시 객체는 **실제 데이터가 필요한 순간에만 DB에서 데이터를 조회** 하도록 설계되어 있다.

### 1️⃣ 프록시 객체가 언제 생성될까?

JPA에서는 `@ManyToOne` 또는 `@OneToOne` 관계에서 `FetchType.LAZY` 를 설정하면 연관된 엔티티를 즉시 조회하지 않고, **프록시 객체만 생성해둔다.**

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)  // 지연 로딩 설정
    @JoinColumn(name = "team_id")
    private Team team;
}
```

- `FetchType.LAZY` 설정을 했기 때문에, `Member` 객체를 조회할 때 `Team` 객체를 **즉시 로딩하지 않고 프록시로 감싸둠.**

### 2️⃣ 프록시 객체란?

JPA는 실제 `Team` 엔티티 대신 프록시 객체(가짜 객체)를 생성해서 `Member.team` 필드에 넣어둠

이 프록시 객체는 실제 데이터를 DB에서 가져오지 않고, 필요할 때(`getter` 호출) 데이터를 가져옴.

```java
// Member 엔티티 조회
Member member = entityManager.find(Member.class, 1L);

// Team 엔티티는 실제로 로딩되지 않음 (프록시 객체)
Team team = member.getTeam();  

// 아직 DB 조회가 일어나지 않음
System.out.println(team.getClass());  
// 출력: class com.sun.proxy.$ProxyXYZ (프록시 객체)

System.out.println(team.getName());  
// 이 순간 DB에서 SELECT 실행!
```

✅ `team.getName()` 을 호출하는 순간, **실제 데이터를 조회하는 SQL이 실행됨!**

### 3️⃣ 프록시 객체가 필요한 이유

✔ **메모리 절약:** 사용하지 않는 데이터는 로딩하지 않아서 **메모리를 아낄 수 있음.**

✔ **성능 최적화:** 필요한 시점에만 데이터를 가져와서 **불필요한 쿼리 실행을 방지.**

✔ **엔티티를 가볍게 유지:** 모든 연관 객체를 한 번에 로딩하면 성능이 저하될 수 있음.

### 4️⃣ 프록시 객체 사용 시 주의할 점

1) **프록시 객체를 영속성 컨텍스트 밖에서 사용하면 예외 발생** (`LazyInitializationException`)

```java
public void getMember() {
    Member member = entityManager.find(Member.class, 1L);
    Team team = member.getTeam();  // 프록시 객체 생성됨
}

// 다른 곳에서 team.getName() 호출하면?
System.out.println(team.getName());  // LazyInitializationException 발생!
```

✅ **해결 방법:**

✔ **트랜잭션 범위 안에서 사용해야 함!**

✔ `JOIN FETCH` 또는 `EntityGraph`를 사용해서 미리 로딩할 수도 있음.