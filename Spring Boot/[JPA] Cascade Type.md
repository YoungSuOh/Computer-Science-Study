## Cascade Type(영속성 전이)

Cascade(영속성 전이)는 **부모 엔티티가 특정 작업(저장, 삭제 등)을 수행할 때, 연관된 자식 엔티티에도 같은 작업을 자동으로 적용하는 기능**이다.

즉, **부모 엔티티를 저장하면 자식 엔티티도 자동으로 저장되고, 부모를 삭제하면 자식도 삭제**되는 등 연관된 엔티티의 생명 주기를 함께 관리할 수 있다.

### 🛠 Cascade Type 종류

| **CascadeType** | **설명** |
| --- | --- |
| ALL | 모든 영속성 전이 적용 (`PERSIST`, `MERGE`, `REMOVE`, `REFRESH`, `DETACH`) |
| PERSIST | 부모 엔티티 저장 시, 자식 엔티티도 자동 저장 (`persist()`) |
| MERGE | 부모 엔티티 변경 시, 자식 엔티티도 자동 병합(`merge()`) |
| REMOVE | 부모 엔티티 삭제 시, 자식 엔티티도 자동 삭제(`remove()`) |
| REFRESH | 부모 엔티티 새로고침 시, 자식 엔티티도 새로고침(`refresh()`) |
| DETACH | 부모 엔티티가 준영속 상태가 될 때, 자식 엔티티도 준영속(`detach()`) |

### Cascade 적용 예제

- `CascadeType.PERSIST` (부모 저장 시 자식 자동 저장)

```java
@Entity
public class Parent {
    @Id @GeneratedValue
    private Long id;
    
    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        children.add(child);
        child.setParent(this);
    }
}

@Entity
public class Child {
    @Id @GeneratedValue
    private Long id;
    
    @ManyToOne
    private Parent parent;
    
    public void setParent(Parent parent) {
        this.parent = parent;
    }
}

```

- 사용 코드

```java
@Transactional
public void saveParentWithChildren() {
    Parent parent = new Parent();
    Child child1 = new Child();
    Child child2 = new Child();

    parent.addChild(child1);
    parent.addChild(child2);

    entityManager.persist(parent); // 부모만 저장해도 자식도 자동 저장됨
}
```

- 출력되는 SQL (`persist()` 실행 시)

```java
INSERT INTO Parent (id) VALUES (1);
INSERT INTO Child (id, parent_id) VALUES (1, 1);
INSERT INTO Child (id, parent_id) VALUES (2, 1);
```

- `persist(parent)`를 호출했을 뿐인데,
- `CascadeType.PERSIST` 덕분에 **자식 엔티티도 자동으로 저장됨.**

- `CascadeType.REMOVE` (부모 삭제 시 자식 자동 삭제)
    
    ```java
    @OneToMany(mappedBy = "parent", cascade = CascadeType.REMOVE)
    private List<Child> children = new ArrayList<>();
    ```
    
- 사용 코드
    
    ```java
    @Transactional
    public void removeParent() {
        Parent parent = entityManager.find(Parent.class, 1L);
        entityManager.remove(parent); // 부모 삭제 시 자식도 삭제됨
    }
    ```
    
- 출력되는 SQL (`remove()` 실행 시)
    
    ```java
    DELETE FROM Child WHERE parent_id = 1;
    DELETE FROM Parent WHERE id = 1;
    ```
    
    - `remove(parent)`를 호출했을 뿐인데,
    - `CascadeType.REMOVE` 덕분에 **자식 엔티티도 자동 삭제됨.**

- `CascadeType.ALL` (모든 영속성 전이)
    
    ```java
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> children = new ArrayList<>();
    ```
    
    - **부모를 저장하면 자식도 저장, 삭제하면 자식도 삭제됨**
    - **`persist()`, `merge()`, `remove()`, `refresh()`, `detach()` 모두 부모와 자식에게 적용됨.**

### 🚀 Cascade 사용 시 주의할 점

✅ **고아 객체(Orphan Removal)와 함께 사용 가능**

- `orphanRemoval = true`를 설정하면 **부모 엔티티에서 관계가 끊긴 자식 엔티티를 자동 삭제**할 수 있다.
    
    ```java
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> children = new ArrayList<>();
    ```
    
    - 부모가 `children.remove(child)`를 호출하면 해당 자식이 DB에서 자동 삭제됨

1️⃣ `CascadeType.ALL`을 신중하게 사용해야 함

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
private List<Child> children = new ArrayList<>();
```

- 설명
    - `CascadeType.ALL`은 **`PERSIST`, `MERGE`, `REMOVE`, `REFRESH`, `DETACH` 모든 작업을 자식에게 전파한다.**
    - 부모 엔티티를 삭제하면 자식 엔티티도 함께 삭제된다.
    - 하지만 연관된 자식 엔티티가 많을 경우, 의도치 않게 대량의 데이터가 삭제될 위험이 있다.
- 주의할 점
    - **정말 부모-자식 관계가 강한 경우만 사용해야 함!** (예: `Order` ↔ `OrderItem`)
    - 다른 엔티티에서도 참조하는 경우 절대 사용하면 안 된다.
    - 예제: 잘못된 사용 예시
        
        ```java
        @Entity
        public class User {
            @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
            private List<Post> posts;
        }
        
        @Entity
        public class Post {
            @ManyToOne
            @JoinColumn(name = "user_id")
            private User user;
        }
        ```
        
        - `User` 삭제 시 **해당 사용자의 모든 `Post`가 삭제됨!**
        - 하지만 `Post`가 다른 곳에서도 사용되는 데이터라면 **치명적인 데이터 손실 발생 가능!**

2️⃣ `CascadeType.REMOVE` 사용 시 데이터 손실 주의

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.REMOVE)
private List<Child> children;
```

- 설명
    - 부모 엔티티 삭제 시, 자식 엔티티도 함께 삭제됨.
    - `CascadeType.ALL`과 유사하지만, **삭제(`REMOVE`)만 전파됨.**
- 주의할 점
    - 자식 엔티티가 다른 엔티티에서도 참조되는 경우 사용하면 안 됨!
    - **외래 키 제약 조건(`ON DELETE CASCADE`)을 고려하여 DB 설계도 함께 검토할 것!**

3️⃣ `CascadeType.MERGE` 사용 시 연관 데이터 갱신 주의

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.MERGE)
private List<Child> children;
```

- 설명
    - 부모 엔티티를 `merge()` 하면, 연관된 자식 엔티티들도 자동으로 `merge()` 된다.
- **주의할 점**
    - 자식 엔티티가 영속성 컨텍스트에 없을 경우, 새로운 엔티티로 인식될 수도 있다.
    - 업데이트 의도가 없는데도 불필요한 쿼리가 실행될 수 있다.
- 예제: 예기치 않은 `merge()` 발생
    
    ```java
    Parent parent = entityManager.find(Parent.class, 1L);
    parent.getChildren().clear();  // 자식 리스트 비우기
    entityManager.merge(parent);   // 불필요한 merge 발생!
    ```
    
    - 자식 리스트가 비워졌으므로, 기존 자식 엔티티가 삭제될 수 있다.
    - **의도치 않은 데이터 손실 가능**

4️⃣ `CascadeType.DETACH` 사용 시 주의할 점

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.DETACH)
private List<Child> children;
```

- 설명
    - 부모 엔티티를 `detach()` 하면, 자식 엔티티도 영속성 컨텍스트에서 제거된다.
    - 이후 변경이 있어도 `flush()` 시점에 반영되지 않다.
- 주의할 점
    - 실수로 `detach()` 호출 시, 데이터가 정상적으로 업데이트되지 않을 수 있다.
    - 대규모 데이터 처리 시 예기치 않은 동작이 발생할 수 있다.
- 잘못된 예제
    
    ```java
    Parent parent = entityManager.find(Parent.class, 1L);
    entityManager.detach(parent);
    parent.getChildren().get(0).setName("변경됨");  // 이 변경은 반영되지 않음!
    entityManager.flush();  // 아무 일도 일어나지 않음
    ```
    
    - 부모를 `detach()` 하면 자식도 `detach`되므로 변경이 반영되지 않다.

5️⃣ `CascadeType.PERSIST` 사용 시 새로운 엔티티 저장 주의

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
private List<Child> children;
```

- 설명
    - 부모 엔티티를 `persist()` 하면, 연관된 자식 엔티티도 함께 `persist()`
- 주의할 점
    - 이미 존재하는 자식 엔티티를 `persist()` 하면 오류 발생 (`Detached entity passed to persist`)
    - 새로운 자식 엔티티만 부모를 통해 `persist()` 해야 한다.
- 잘못된 예제
    
    ```java
    Child existingChild = entityManager.find(Child.class, 1L);
    Parent parent = new Parent();
    parent.getChildren().add(existingChild);
    entityManager.persist(parent);  // 기존 Child를 persist하려 해서 오류 발생!
    ```
    
    - 기존 엔티티를 `persist()` 하면 `org.hibernate.PersistentObjectException` 발생