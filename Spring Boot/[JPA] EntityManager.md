# [JPA] EntityManager

## JPA에서 EntityManager 란?

`EntityManager`는 JPA의 핵심 인터페이스로, **엔티티(객체)와 데이터베이스 사이의 상호작용을 관리**하는 역할

JPA에서는 SQL을 직접 다루는 것이 아니라 **객체 지향 방식**으로 데이터를 관리하는데, `EntityManager`가 이를 담당한다.

## EntityManager의 역할

1. 트랜잭션 관리
2. 영속성 컨텍스트 관리
3. 쿼리 실행
4. 캐시 관리

## JPA에서 EntityManager의 주요 메서드

### **1. 엔티티 저장 (`persist`)**

데이터베이스에 **새로운 엔티티를 저장**합니다.

`persist()`를 호출하면 엔티티가 **영속성 컨텍스트에 저장**되며, 트랜잭션이 커밋될 때 DB에 반영됩니다.

```java
Member member = new Member();
member.setName("John Doe");

entityManager.persist(member); // 영속성 컨텍스트에 저장
```

### **2. 엔티티 조회 (`find` & `getReference`)**

1. **`find(Class<T> entityClass, Object primaryKey)`**
    - **즉시 조회(Eager Loading)**: DB에서 데이터를 바로 가져오거나 1차 캐시에서 조회
    - **실제 엔티티 객체 반환**
        
        ```java
        Member member = entityManager.find(Member.class, 1L);
        ```
        

1. `getReference(Class<T> entityClass, Object primaryKey)`
    - **지연 로딩(Lazy Loading)**: 실제 조회가 필요할 때 데이터를 가져옴
    - **프록시 객체 반환** → 실제 DB 조회 전까지 쿼리 실행 X
        
        ```java
        Member member = entityManager.getReference(Member.class, 1L);
        System.out.println(member.getName()); // 이때 쿼리 실행됨 (Lazy Loading)
        ```
        

1. 엔티티 수정 (`merge`)
    - Detached 상태의 엔티티를 영속 상태로 변경(갱신)할 때 사용
    - 엔티티를 `merge()`하면 새로운 영속 상태의 엔티티가 반환되며, 변경 사항이 자동으로 반영
        
        ```java
        Member detachedMember = new Member();
        detachedMember.setId(1L);
        detachedMember.setName("Updated Name");
        
        Member mergedMember = entityManager.merge(detachedMember);
        ```
        

1. **엔티티 삭제 (`remove`)**
    - 영속 상태의 엔티티를 삭제
    - 삭제된 엔티티는 **트랜잭션이 커밋될 때 실제로 DB에서 제거**
        
        ```java
        Member member = entityManager.find(Member.class, 1L);
        entityManager.remove(member);
        ```
        

1. 쿼리 실행 (`createQuery`, `createNativeQuery`)
    - JPQL (객체 기반 쿼리)
        
        ```java
        List<Member> members = entityManager.createQuery("SELECT m FROM Member m WHERE m.name = :name", Member.class)
                                            .setParameter("name", "John Doe")
                                            .getResultList();
        
        ```
        
    - 네이티브 SQL 실행
        
        ```java
        List<Object[]> resultList = entityManager.createNativeQuery("SELECT * FROM Member WHERE name = ?", Member.class)
                                                 .setParameter(1, "John Doe")
                                                 .getResultList();
        
        ```
        

1. 트랜잭션 관리 (`getTransaction`)
    - 트랜잭션을 직접 관리할 때 사용
        
        ```java
        entityManager.getTransaction().begin();
        
        Member member = new Member();
        member.setName("New User");
        entityManager.persist(member);
        
        entityManager.getTransaction().commit();
        
        ```