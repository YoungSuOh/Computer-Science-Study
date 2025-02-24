# [JPA] Persistence Context

## JPA & Hibernates

### 차이점

**JPA는 표준 API이고 Hibernate는 구현체**

| **비교 항목** | JPA (Java Persistence API) | Hibernate |
| --- | --- | --- |
| 정의 | Java 진영에서 만든 ORM 표준 명세 | JPA의 구현체 중 하나 (가장 많이 사용됨) |
| 역할 | ORM을 위한 인터페이스 제공 (구현 X) | JPA의 기능을 실제로 구현 |
| **SQL 자동 생성** | 지원 (하지만 직접 SQL 실행은 불가능) | 지원 + 더 많은 기능 제공 (예: `@Filter`, `@LazyCollection`) |
| **캐시 기능** | 1차 캐시 제공 | 1차 + 2차 캐시 제공 |
| **쿼리 언어** | JPQL 사용 | JPQL + HQL(Hibernate Query Language) |
| **스프링에서 주로 사용하는 방식** | `Spring Data JPA`와 함께 사용 | `Spring Data JPA + Hibernate` 조합으로 사용 |

### Hibernate에서 Session과 Transaction의 역할

Hibernate에서 `Session`은 **데이터베이스와의 연결을 관리하는 객체**

```java
Session session = sessionFactory.openSession();
Transaction transaction = session.beginTransaction();

User user = new User("홍길동");
session.save(user);  // INSERT 실행
transaction.commit();
session.close();
```

Hibernate의 `Transaction`은 **하나의 논리적인 작업 단위를 보장하는 객체**

```java
Session session = sessionFactory.openSession();
Transaction transaction = session.beginTransaction();

try {
    User user = session.get(User.class, 1L);  // SELECT 실행
    user.setName("이순신");
    session.update(user);  // UPDATE 실행

    transaction.commit();  // 변경 사항 반영
} catch (Exception e) {
    transaction.rollback();  // 오류 발생 시 롤백
} finally {
    session.close();
}
```

**정리**

| 개념 | JPA | Hibernate |
| --- | --- | --- |
| ORM 표준 | O | X (JPA 구현체) |
| Session | `EntityManager` | `Session` |
| Transaction | `EntityTransaction` | `Transaction` |
| **영속성 컨텍스트** | `EntityManager`에 의해 관리 | `Session`이 1차 캐시 역할 수행 |

## JPA 영속성 컨텍스트 (Persistence Context)

![image](https://github.com/user-attachments/assets/b0427134-73a7-4df6-bed1-b12b8295c24c)

영속성 컨텍스트(Persistence Context)는 엔티티를 저장하고 관리하는 **메모리 공간이다**.

쉽게 말하면, JPA에서 데이터베이스와 상호작용할 때 **객체를 캐시처럼 관리하는 영역**이라고 생각하면 된다.

- 특징
    1. 엔티티의 생명주기 관리
    2. 1차 캐시 제공
    3. 변경 감지 (Dirty Checking)
    4. 쓰기 지연 (Write-Behind, Delayed Write)

### EntityManager 객체 생성과 Entity 클래스 정의

![image (1)](https://github.com/user-attachments/assets/ea3c98cb-8909-46f8-b0b1-7db565d2c3d6)


1. `persistence.xml` 파일을 통해서 JPA 관련 정보를 설정한다.
    1. resources의 META-INF에 추가
        
        ![image (2)](https://github.com/user-attachments/assets/5786f28d-4cb9-408e-b0f2-d7037947a749)

        
    2. 영속성 컨텍스트에서 관련 SQL을 데이터베이스에 전달할 때는 데이터베이스별 Dialect(방언)을 적용한다.
    
    ![image (3)](https://github.com/user-attachments/assets/2ecde4f7-2a86-46e2-bef3-8e96124e9e69)

    
    ```java
    <?xml version="1.0" encoding="UTF-8"?>
    <persistence xmlns="https://jakarta.ee/xml/ns/persistence"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd"
                 version="3.0">
    
        <persistence-unit name="entitytest"> // 영속성 
            <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>  // JPA 구현체로 Hibernate를 사용
            <class>org.example.jpatest.exam.EntityTest01</class> <!--매니저가  관리할 수 있도록 알림-->
            <properties>
                <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://db-2vm3fl-kr.vpc-pub-cdb.ntruss.com:3306/bootdb"/>
                <property name="jakarta.persistence.jdbc.user" value="youngsu"/>
                <property name="jakarta.persistence.jdbc.password" value="qwe123!@#"/>
    
    						// dialect 설정
                **<property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>**
                <property name="hibernate.show_sql" value="true"/> // 쿼리를 콘솔에 출력
                <property name="hibernate.format_sql" value="true"/>
                <property name="hibernate.use_sql_comments" value="true"/>
                <property name="hibernate.hbm2ddl.auto" value="update"/>
            </properties>
        </persistence-unit>
    </persistence>
    ```
    
2. EntityManagerFactory 객체를 생성한다. ( <persistence-unit> 태그의 이름 정보 지정)
    
    ![image (4)](https://github.com/user-attachments/assets/7ff83d5b-d9e6-4cb3-846e-7b9c8f3b46a9)

    
3. EntityManager 객체를 생성하여 Entity 객체를 영속성 컨텍스트를 통해 관리한다.
    
    ```java
    public class EntityTestApp01 {
        public static void main(String[] args) {
            EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("entitytest");
            EntityManager entityManager = entityManagerFactory.createEntityManager();
            entityManager.getTransaction().begin(); // 트랜잭션 시작
            EntityTest01 entityTest01;
    
            for(int i=1;i<=5;i++){
                entityTest01 = new EntityTest01();
                entityTest01.setName("홍길동"+i);
                entityTest01.setAge((int) (Math.random()*20+20));
                entityTest01.setBirthday(LocalDateTime.now());
                entityManager.persist(entityTest01);
            }
    
            // persist는 영속성 컨텍스트로
            entityManager.getTransaction().commit(); // DML commit 시켜줌
            entityManager.close();
            entityManagerFactory.close();
        }
    }
    ```
    
    
    ![image (5)](https://github.com/user-attachments/assets/ee75622b-b0a4-4cd6-b7a1-b87d36ee86ad)


### JPA에서 Entity의 Life Cycle (엔티티의 생명주기)

![image (6)](https://github.com/user-attachments/assets/2b30f557-89df-4c94-aeb8-5f126c4537ee)


JPA에서 엔티티(Entity)는 특정 상태를 가지며, 상태에 따라 JPA가 관리하는 방식이 달라진다.

엔티티는 비영속, 영속, 준영속, 삭제 이렇게 4가지로 나뉜다.

| **상태** | **설명** | **예제** |
| --- | --- | --- |
| **비영속 (New, Transient)** | JPA가 관리하지 않는 상태 | **`new Member("홍길동")`** |
| **영속 (Managed, Persistent)** | 영속성 컨텍스트에 저장된 상태, 변경 감지됨 | `em.persist(member)` |
| **준영속 (Detached)** | 영속성 컨텍스트에서 분리된 상태, 변경 감지 안됨 | `em.detach(member)` |
| **삭제 (Removed)** | 영속성 컨텍스트에서 삭제된 상태 | `em.remove(member)` |

### 엔티티 상태 전이 과정

엔티티의 상태는 **JPA의 메서드 호출 및 트랜잭션 처리에 따라 전이된다,**

```java
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// (1) 비영속 상태
Member member = new Member();
member.setName("홍길동");  // 아직 JPA가 관리하지 않음.

// (2) 영속 상태
em.persist(member); 
// JPA가 1차 캐시에 저장하고, 변경 감지(Dirty Checking) 기능을 적용
// 트랜잭션이 commit()될 때 flush()가 실행되어 DB에 반영

// 아직 INSERT SQL 실행 안 됨 (쓰기 지연)
em.getTransaction().commit(); // 트랜잭션 커밋 시 INSERT 실행

// (3) 준영속 상태
em.detach(member);
// detach(entity), clear(), close()를 호출하면 영속성 컨텍스트에서 분리
// 더 이상 변경 감지가 동작하지 않음.
// 트랜잭션이 커밋되어도 변경 사항이 반영되지 않음.

// (4) 다시 영속 상태로 변경
Member mergedMember = em.merge(member);

// (5) 삭제 상태
em.remove(mergedMember);
// remove(entity)를 호출하면 삭제 상태가 됨
// 트랜잭션이 commit()될 때 DELETE SQL 실행.

em.getTransaction().commit();

em.persist(member);  // INSERT SQL이 생성되지만 즉시 실행되지 않음 (쓰기 지연)
em.flush();          // INSERT SQL이 즉시 실행됨
// flush()를 호출하면 변경 감지(Dirty Checking)와 SQL 실행이 즉시 발생.
// 트랜잭션이 유지되는 한, flush 이후에도 엔티티는 여전히 영속 상태로 남아 있음. -> 롤백 가능
```

### JPA 1차 캐시

- **영속성 컨텍스트(EntityManager)에 저장되는 캐시**
- 같은 엔티티를 **반복 조회해도 DB를 조회하지 않고 캐시에서 가져옴**
- **트랜잭션 범위 내에서만 유효함** (트랜잭션 종료 시 사라짐)

```java
// 트랜잭션 시작
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// ID가 1인 Member 엔티티 조회 (DB에서 가져와서 1차 캐시에 저장됨)
Member member1 = em.find(Member.class, 1L);

// 같은 ID로 다시 조회 (DB 조회 X, 1차 캐시에서 가져옴)
Member member2 = em.find(Member.class, 1L);

System.out.println(member1 == member2); // true (같은 객체)
```

- 처음 `find()`를 호출하면 DB에서 조회 후 1차 캐시에 저장
- 이후 같은 엔티티를 조회하면 DB를 조회하지 않고 1차 캐시에서 가져옴
- 트랜잭션이 종료되면 1차 캐시는 삭제됨
    - `commit()` 또는 `rollback()` 후에는 캐시가 초기화됨
- 엔티티를 수정하면 자동 반영 (변경 감지)
    - 1차 캐시 덕분에 **Dirty Checking(변경 감지)이 가능**

### JPA에서 Dirty Checking (변경 감지)

**Dirty Checking(변경 감지)**는 **JPA가 영속성 컨텍스트(Persistence Context) 내에서 관리하는 엔티티의 변경 사항을 자동으로 감지하여 DB에 반영하는 기능**

즉, 개발자가 `update` SQL을 직접 작성하지 않아도, **트랜잭션이 커밋될 때 변경된 엔티티를 자동으로 업데이트**해준다.

**동작하는 방식**

1. **엔티티 조회 (영속 상태)**
    - `find()` 또는 `JPQL`을 사용하여 엔티티를 조회하면, **1차 캐시에 저장됨.**
2. **스냅샷(Snapshot) 저장**
    - JPA는 **처음 조회한 엔티티의 상태를 스냅샷으로 저장**해둠.
    - 변경 전 상태를 기억해 둠.
3. **엔티티 값 변경**
    - 엔티티의 값을 setter 또는 직접 변경하여 수정.
4. **트랜잭션 커밋 시 비교**
    - 트랜잭션이 `commit()`될 때, **현재 엔티티와 스냅샷을 비교**함.
    - 변경된 데이터가 있다면, **자동으로 `update` SQL을 실행**하여 DB에 반영.
5. **DB 업데이트**
    - 변경된 필드만 포함한 `UPDATE` 쿼리가 실행됨.

```java
@Transactional
public void updateMember(Long id) {
    // 1. 영속성 컨텍스트에서 엔티티 조회 (1차 캐시에 저장됨)
    Member member = entityManager.find(Member.class, id);

    // 2. 엔티티 필드 값 변경 (Setter를 이용)
    member.setName("홍길동");
    member.setAge(30);

    // 3. 트랜잭션이 커밋될 때 Dirty Checking이 실행됨
    // 변경된 값이 감지되면 update 쿼리가 자동 실행됨
}

```

✅ **출력되는 SQL (Dirty Checking으로 자동 생성됨)**

```java
UPDATE Member SET name = '홍길동', age = 30 WHERE id = 1;
```

🔹 **Dirty Checking을 비활성화하려면?**

- **Dirty Checking을 원하지 않는 경우**에는 `@Transactional(readOnly = true)`를 사용하여 변경 감지를 막을 수 있다.

```java
@Transactional(readOnly = true)
public Member getMember(Long id) {
    return entityManager.find(Member.class, id);
}
```

### JPA에서 쓰기 지연 (Write-Behind, Delayed Write)

- 엔티티 변경(INSERT, UPDATE, DELETE) 작업을 즉시 데이터베이스에 반영하지 않고, 트랜잭션이 커밋될 때 한꺼번에 SQL을 실행하는 최적화 기법이다.

- 🛠 쓰기 지연이 동작하는 방식
    1. 트랜잭션 시작 → 엔티티 저장 요청
        - `persist()`, `merge()`, `remove()`를 호출하면 엔티티 매니저가 변경 사항을 영속성 컨텍스트(1차 캐시)에 저장한다.
        - 즉시 `INSERT`, `UPDATE`, `DELETE` 쿼리를 실행하지 않고, **쓰기 지연 SQL 저장소(쓰기 지연 큐, SQL 저장소)에 쿼리를 보관해 둔다.**
    2. 변경된 엔티티는 1차 캐시에 저장됨
        - JPA는 먼저 **1차 캐시에 데이터를 저장하고**, 데이터베이스에는 바로 반영하지 않는다.
    3. 트랜잭션 커밋 시점에 쓰기 지연 SQL 실행
        - `commit()`이 호출되면, JPA는 **쓰기 지연 SQL 저장소에 모아둔 SQL을 한 번에 실행**해서 성능을 최적화함.
        - **트랜잭션 내에서 여러 번 변경해도 DB 호출이 최소화됨.**

- 쓰기 지연의 장점
    1. 트랜잭션 내에서 여러 번 변경해도 DB 호출 최소화
        - 여러 개의 `persist()`, `merge()`, `remove()`를 호출해도 **DB에 즉시 반영되지 않고, 트랜잭션이 끝날 때 한 번에 실행**됨.
        - 성능이 향상되고 불필요한 DB I/O를 줄일 수 있음
    2. Batch 처리로 최적화 가능
        - DB에 데이터를 저장할 때 **한 번에 여러 개의 `INSERT` 또는 `UPDATE`를 실행**하는 것이 가능함.
        - 대량 데이터 처리에서 유용함.
    3. JPA가 자동으로 변경 사항을 감지하고 최적화
        - 같은 엔티티에 여러 번 변경이 발생해도 **최종 변경 사항만 반영되므로 중복 SQL 실행이 방지됨.**

- **Dirty Checking:** 트랜잭션이 끝날 때 변경된 엔티티를 감지하고 자동으로 `UPDATE` 실행.
- **쓰기 지연:** `UPDATE`, `INSERT`, `DELETE` 같은 SQL을 모아뒀다가 **트랜잭션이 끝날 때 실행.**
