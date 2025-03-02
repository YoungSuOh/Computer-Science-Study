# [JPA] 연관 관계 매핑

## @OneToOne 양방향 연관 관계

### 특징

- 한 개의 엔티티가 **다른 하나의 엔티티와 1:1 관계**를 가짐.
- 주 테이블의 **외래 키를 어느 테이블에 둘 것인지** 결정해야 함.
- 주인이 아닌 쪽에는 `mappedBy`를 설정.

### 예제 코드

```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    
    private String name;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    private Address address;
}

@Entity
public class Address {
    @Id @GeneratedValue
    private Long id;

    private String city;
    private String street;

    @OneToOne
    @JoinColumn(name = "user_id") // 외래 키를 Address 테이블이 관리
    private User user;
}
```

**생성되는 테이블**

| User 테이블 | Address 테이블 |
| --- | --- |
| id (PK) | id (PK) |
| name | city |
|  | street |
|  | user_id (FK, UNIQUE)  |

**설명**

✅ `Address`가 외래 키(`user_id`)를 가지므로 **연관 관계의 주인**

✅ `User`는 `mappedBy = "user"`로 주인이 아님을 표시

## @OneToMany / @ManyToOne 양방향 연관 관계

### 특징

- **일대다(OneToMany): 한 개의 엔티티가 여러 개의 엔티티를 가짐**
- **다대일(ManyToOne): 여러 개의 엔티티가 하나의 엔티티를 참조**
- 다대일(`@ManyToOne`)이 **외래 키를 가지며 연관 관계의 주인**
- **일대다(`@OneToMany`)는 `mappedBy`를 설정해야 함**

### 예제 코드

```java
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team", cascade = CascadeType.ALL)
    private List<Member> members = new ArrayList<>();
}

@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "team_id") // Member 테이블이 외래 키를 관리
    private Team team;
}

```

**예시 데이터**

```java
Team 테이블
------------------
id | name
1  | Team A
2  | Team B

Member 테이블
------------------
id | username | team_id
1  | Alice    | 1
2  | Bob      | 1
3  | Charlie  | 2
```

**설명**

✅ `Member`가 `team_id` 외래 키를 가지므로 **연관 관계의 주인**

✅ `Team`은 `mappedBy = "team"`로 주인이 아님을 표시

## @ManyToMany 양방향 연관 관계

### 특징

- 다대다 관계는 **중간 테이블(연결 테이블)이 자동 생성됨.**
- 실무에서는 **다대다를 직접 사용하지 않고, 중간 엔티티를 만들어 다대일-일대다 관계로 푸는 것이 일반적!**
- 주 테이블에 `@JoinTable`을 사용하여 중간 테이블을 매핑.

```java
@Entity
public class Student {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_course", // 중간 테이블 이름
        joinColumns = @JoinColumn(name = "student_id"), // 현재 엔티티(Student)의 외래 키
        inverseJoinColumns = @JoinColumn(name = "course_id") // 상대 엔티티(Course)의 외래 키
    )
    private List<Course> courses = new ArrayList<>();
}

@Entity
public class Course {
    @Id @GeneratedValue
    private Long id;

    private String title;

    @ManyToMany(mappedBy = "courses") // 주인이 아님을 표시
    private List<Student> students = new ArrayList<>();
}

```

**생성되는 테이블**

| Student 테이블 | Course 테이블 | Student_Course 테이블 (연결 테이블) |
| --- | --- | --- |
| id (PK) | id (PK) | student_id (FK) |
| name | title | course_id (FK) |
- **다대다 관계에서는 중간 테이블(`student_course`)이 자동 생성됨**
- 중간 테이블이 **두 개의 외래 키(student_id, course_id)**를 가짐
- 학생과 강의가 **N:M 관계를 맺음**

**예시 데이터**

```java
Student 테이블
------------------
id | name
1  | Alice
2  | Bob

Course 테이블
------------------
id | title
1  | Math
2  | Science

Student_Course 테이블
----------------------
student_id | course_id
1          | 1  (Alice -> Math)
1          | 2  (Alice -> Science)
2          | 1  (Bob -> Math)
```

**설명**

✅ `Student`가 `@JoinTable`을 사용하여 중간 테이블을 관리

✅ `Course`는 `mappedBy = "courses"`로 주인이 아님을 표시

## `mappedBy` 의 역할

- **양방향 연관 관계에서 연관 관계의 주인을 지정하는 속성**
- `mappedBy`가 설정된 쪽은 **연관 관계의 주인이 아니며, 외래 키를 관리하지 않음**
- **반대로 연관 관계의 주인은 `@JoinColumn`을 사용하여 외래 키를 직접 관리함**

1. `mappedBy`가 없는 경우 (양방향 관계 아님)
    
    ```java
    @Entity
    public class Team {
        @Id @GeneratedValue
        private Long id;
        private String name;
    }
    
    @Entity
    public class Member {
        @Id @GeneratedValue
        private Long id;
        private String username;
    
        @ManyToOne
        @JoinColumn(name = "team_id") // FK를 직접 관리
        private Team team;
    }
    ```
    
    📌 결과 테이블
    
    | Team 테이블 | Member 테이블 |
    | --- | --- |
    | id (PK) | id (PK) |
    | name | username |
    |  | team_id (FK) |
    - **`Member` 엔티티가 `team_id` 외래 키를 직접 관리**
    - **이 경우 Team 입장에서 Member를 조회하는 방법이 없음 → 양방향 아님**
2. `mappedBy`를 사용한 양방향 관계
    
    ```java
    @Entity
    public class Team {
        @Id @GeneratedValue
        private Long id;
        private String name;
    
        @OneToMany(mappedBy = "team") // 연관 관계의 주인이 아님 (읽기 전용)
        private List<Member> members = new ArrayList<>();
    }
    
    @Entity
    public class Member {
        @Id @GeneratedValue
        private Long id;
        private String username;
    
        @ManyToOne
        @JoinColumn(name = "team_id") // 연관 관계의 주인 (FK 관리)
        private Team team;
    }
    ```
    
    📌 결과 테이블
    
    | Team 테이블 | Member 테이블 |
    | --- | --- |
    | id (PK) | id (PK) |
    | name | username |
    |  | team_id (FK) |
    
    **📌 `mappedBy = "team"` 설정 의미**
    
    - **`Member` 엔티티가 `team_id` FK를 직접 관리하는 "연관 관계의 주인"**
    - **`Team` 엔티티는 단순히 `mappedBy`로 관계를 읽기만 할 수 있음 (조회 전용)**
    - **DB 테이블 구조에는 아무 변화 없음, 외래 키는 여전히 `Member` 테이블이 관리**
3. 🚨 `mappedBy`를 사용하지 않은 경우 발생하는 문제
    - 문제 1: JPA가 관계를 두 번 관리하여 데이터 불일치 발생
        - 양방향 관계에서 `mappedBy` 없이 **양쪽 모두 FK를 관리하려고 하면** JPA는 서로 독립적인 관계로 판단해서 중복된 데이터가 저장될 수 있다.
        
        ```java
        @Entity
        public class Team {
            @Id @GeneratedValue
            private Long id;
            private String name;
        
            @OneToMany // mappedBy 없이 직접 관리
            @JoinColumn(name = "team_id") 
            private List<Member> members = new ArrayList<>();
        }
        
        @Entity
        public class Member {
            @Id @GeneratedValue
            private Long id;
            private String username;
        
            @ManyToOne
            @JoinColumn(name = "team_id") 
            private Team team;
        }
        
        ```
        
        - `@OneToMany` 쪽에서 `@JoinColumn(name = "team_id")`를 설정했는데, 사실상 `Member` 테이블에서 `team_id`를 직접 관리하고 있음
        - 하지만 `Member` 엔티티에서도 `@ManyToOne`을 통해 같은 `team_id`를 관리하고 있다.
        - 이러면 JPA는 같은 외래 키를 두 번 관리하게 된다.
    - 문제 2 : 중복된 데이터 저장
        
        ```java
        Team team = new Team();
        team.setName("Team A");
        
        Member member1 = new Member();
        member1.setUsername("User1");
        member1.setTeam(team); // team_id 저장
        
        Member member2 = new Member();
        member2.setUsername("User2");
        member2.setTeam(team); // team_id 저장
        
        team.getMembers().add(member1);
        team.getMembers().add(member2);
        
        // 부모, 자식 모두 persist
        em.persist(team);
        em.persist(member1);
        em.persist(member2);
        
        ```
        
        - JPA는 @OneToMany에서도 `team_id`를 관리하려 하고, `@ManyToOne`에서도 `team_id` 를 관리하려 하기 때문에 데이터가 중복 저장되거나, JPA가 어떤 FK가 “진짜”인지 혼란스러워함.

## Cascade 옵션

- 부모 엔티티가 **변경될 때 자식 엔티티도 함께 변경**되도록 설정하는 옵션
- 주로 `@OneToMany`, `@OneToOne` 관계에서 사용

### `cascade` 옵션 종류

| **옵션** | **설명** |
| --- | --- |
| **`CasadeType.PESIST`** | 부모 엔티티를 저장할 때, 자식 엔티티도 함께 저장 |
| **`CasadeType.REMOVE`** | 부모 엔티티를 삭제할 때, 자식 엔티티도 함께 삭제 |
| **`CascadeType.ALL`** | 모든 상태 변화(`PERSIST`, `REMOVE` 등)를 전파 |
| `CascadeType.MERGE` | 부모 엔티티를 병합(갱신)할 때, 자식 엔티티도 병합 |
| `CascadeType.DETACH` | 부모 엔티티가 detach(영속성 컨텍스트에서 분리)될 때, 자식도 함께 분리 |
| `CascadeType.REFRESH` | 부모 엔티티를 DB에서 다시 읽어올 때, 자식도 함께 새로 고침 |

### `cascade` 예제

```java
@Entity
public class Parent {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> children = new ArrayList<>();
}

@Entity
public class Child {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;
}

```

**📌 `cascade = CascadeType.ALL` 설정 의미**

- **부모 `Parent`를 저장하면 `Child`도 자동으로 저장됨**
- **부모 `Parent`를 삭제하면 `Child`도 자동으로 삭제됨**

### `cascade` 옵션이 없는 경우

```java
Parent parent = new Parent();
Child child1 = new Child();
Child child2 = new Child();

parent.getChildren().add(child1);
parent.getChildren().add(child2);

// 부모, 자식 따로 저장해야 함 (에러 발생 가능)
entityManager.persist(parent);
entityManager.persist(child1);
entityManager.persist(child2);
```

- **`cascade` 없으면 부모-자식 각각 `persist()` 호출해야 함**
- **실수로 자식 엔티티를 저장 안 하면, 부모와 연관된 자식이 없는 상태가 될 수도 있음**

### `cascade` 적용한 경우

```java
Parent parent = new Parent();
Child child1 = new Child();
Child child2 = new Child();

parent.getChildren().add(child1);
parent.getChildren().add(child2);

// 부모만 persist하면 자동으로 자식도 저장됨
entityManager.persist(parent);
```

- 부모만 저장해도 `CascadeType.PERSIST`가 전파되어 자식도 자동 저장됨!