# JPA의 탄생 배경

## JDBC

- JDBC는 Java에서 데이터 베이스에 접속하고 데이터 처리구현에 사용되는 Java API이다.
- **장점**: 다양한 DBMS에서 동일한 JDBC API를 이용할 수 있어 **DBMS에 종속되지 않는다.**
- 그러나 JDBC를 직접 사용하면 여러 가지 단점이 있다.

### ❌ JDBC의 단점

1. **SQL 작성과 객체 변환을 직접 해야 함**
    - `ResultSet`을 이용해 데이터를 꺼낸 후, 객체에 직접 매핑해야 해서 **반복 코드가 많아진다.**
2. **트랜잭션 및 자원 관리 부담**
    - 커넥션, 스테이트먼트, 리소스를 직접 관리해야 하므로 **자원 누수 위험**이 있다.
3. **SQL 재사용 어려움**
    - 같은 기능의 SQL이라도 다른 곳에서 또 작성해야 해서 유지보수가 어렵다.

### 사용 예시

```java
public class BoardDAO {
    private final String driver = "oracle.jdbc.driver.OracleDriver";
    private final String url = "jdbc:oracle:thin:@localhost:1521:XE";
    private final String user = "C##java";
    private final String password = "oracle";

    private Connection conn;
    private PreparedStatement pstmt;
    private ResultSet rs;

    private static BoardDAO instance = new BoardDAO();

    public BoardDAO() {
        try{
            Class.forName(driver);
            System.out.println("loaded driver");
        }catch(ClassNotFoundException e){
            e.printStackTrace();
        }
    }
    public static BoardDAO getInstance() {  // 싱글톤
        return instance;
    }

    public void getConnection(){
        try {
            conn = DriverManager.getConnection(url,user,password);
        }catch(SQLException e){
            e.printStackTrace();
        }
    }

    public void closeConnection(){
        try{
            if(rs != null){rs.close();}
            if(pstmt!=null){pstmt.close();}
            if(conn!=null){conn.close();}
            System.out.println("connection closed");
        }catch(SQLException e){
            e.printStackTrace();
        }
    }

    public void writeBoard(BoardDTO boardDTO) {
        this.getConnection();
        try{
            pstmt=conn.prepareStatement("insert into board_java values (board_java_seq.NEXTVAL,?,?,?,?,sysdate)");
            pstmt.setString(1,boardDTO.getId());
            pstmt.setString(2,boardDTO.getName());
            pstmt.setString(3,boardDTO.getSubject());
            pstmt.setString(4,boardDTO.getContent());

            int update = pstmt.executeUpdate();
            if(update>0){
                System.out.println("작성하신 글을 등록하였습니다.");
            }
            else{
                System.out.println("작성하신 글 등록에 실패하였습니다.");
            }

        }catch(SQLException e){
            e.printStackTrace();
        }
        closeConnection();

    }

    public void listBoard() {
        System.out.println("---------------------------------------------------------\n" +
                "글번호\t제목\t\t아이디\t날짜\n" +
                "---------------------------------------------------------");
        this.getConnection();
        try{
            pstmt=conn.prepareStatement("select * from board_java ORDER BY LOGTIME DESC");

            rs=pstmt.executeQuery();

            while(rs.next()){
                System.out.println(rs.getInt(1)+"\t"+rs.getString(4)+
                        "\t"+rs.getString(2)+"\t"+rs.getDate(6));
            }

        }catch(SQLException e){
            e.printStackTrace();
        }
        closeConnection();

    }

    public void viewBoard() {
        System.out.print("글 번호 입력 : ");
        Scanner scanner = new Scanner(System.in);
        int num = scanner.nextInt();

        this.getConnection();
        try{
            pstmt=conn.prepareStatement("select * from board_java where seq=?");
            pstmt.setInt(1,num);

            rs=pstmt.executeQuery();

            while(rs.next()){
                System.out.println("아이디 : "+rs.getString(2));
                System.out.println("이름 : "+rs.getString(3));
                System.out.println("날짜 : "+rs.getDate(6));
                System.out.println("내용 : "+rs.getString(5));
            }

        }catch(SQLException e){
            e.printStackTrace();
        }
        closeConnection();

    }

}

```

## SQL Mapper

- **SQL Mapper는 직접 작성한 SQL과 객체를 매핑하는 방식**의 퍼시스턴스 프레임워크다.
- 즉, SQL을 실행한 결과를 개발자가 정의한 객체의 필드와 매핑한다.
- 대표적인 SQL Mapper: **MyBatis**

### ✅ 장점

- SQL과 객체 매핑을 자동화하여 **코드를 단순화**할 수 있다.
- JDBC보다 **유지보수성이 뛰어나고 중복 코드가 줄어든다.**
- SQL 튜닝이 용이하여 복잡한 쿼리 작성이 가능하다.

### ❌ 단점

1. **SQL을 직접 작성해야 하므로 특정 DB에 종속적**
2. **CRUD 코드 반복** (비슷한 SQL을 여러 번 작성해야 함)
3. **테이블 필드 변경 시 유지보수 어려움**
    - SQL과 객체 매핑을 수동으로 해야 하므로, 테이블 변경 시 SQL도 수정해야 한다.

### 사용 예시

- [db.properties](http://db.properties) → yml임
    
    ```java
    jdbc.driver=oracle.jdbc.driver.OracleDriver
    jdbc.url=jdbc:oracle:thin:@localhost:1521:xe
    jdbc.username=c##java
    jdbc.password=oracle
    ```
    
- mybatis-config.xml
    
    ```java
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration PUBLIC "//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd" >
    <configuration>
    	<properties resource="db.properties"></properties>
    
    	<typeAliases>
    		<typeAlias type="user.bean.UserDTO" alias="user"/>
    	</typeAliases>
    
    	<environments default="development">
    		<environment id="development">
    			<transactionManager type="JDBC" />
    			<dataSource type="POOLED">
    				<property name="driver" value="${jdbc.driver}"/>
    				<property name="url" value="${jdbc.url}"/>
    				<property name="username" value="${jdbc.username}"/>
    				<property name="password" value="${jdbc.password}"/>
    			</dataSource>
    		</environment>
    	</environments>
    	<mappers>
    		<mapper resource="mapper/userMapper.xml"/>
    	</mappers>
    </configuration>
    ```
    
    - `<properties resource="db.properties"></properties>` 로 [db.properties](http://db.properties)  정보들 받음
    - `transactionManager`가 `JDBC`로 설정되어 있으므로, MyBatis가 데이터베이스 연결에 직접 트랜잭션을 관리한다.
- UserDAO
    
    ```java
    public List<UserDTO> getAllList() {
    	      SqlSession sqlSession = sqlSessionFactory.openSession();   //생성
    	      List<UserDTO> list = sqlSession.selectList("userSQL.getAllList");
    	      sqlSession.close();
    	      return list;
    }
    ```
    
- userMapper.xml
    
    ```java
    <resultMap type="user" id="userResult">
    		<result column="NAME" property="name" />
    		<result column="ID" property="id" />
    		<result column="PWD" property="pwd" />
    	</resultMap>
    	
    	<select id="getAllList" resultMap="userResult">
    		select * from usertable
    	</select>
    ```
    

## ORM

- ORM(Object-Relational Mapping)은 **객체와 RDB 테이블을 자동으로 매핑하는 기술**이다.
- ORM을 사용하면 SQL을 직접 작성하지 않고 **객체의 메서드를 호출하는 것만으로 DB 조작 가능**
- Java에서 대표적인 ORM 기술: **JPA(Java Persistence API)**

### ✅ ORM의 장점

1. **SQL을 자동 생성**하여 개발자가 직접 작성할 필요가 없음
2. **DB 변경 시에도 코드 수정 최소화**
3. **객체지향적인 개발 가능** (객체 간 연관 관계를 그대로 사용 가능)
4. **1차 캐시, 지연 로딩, 변경 감지 기능 제공** (성능 최적화 가능)

## Hibernate

Hibernate는 Java의 ORM(Object-Relational Mapping) 프레임워크로, 객체와 관계형 데이터베이스(RDB)를 쉽게 매핑하여 사용할 수 있도록 도와주는 강력한 도구이다.

### 주요 기능

1. SQL 자동 생성
2. 데이터베이스 독립성
    - 특정 데이터베이스 벤더(MySQL, PostgreSQL, Oracle 등)에 의존하지 않고, **Dialect(방언)** 설정만 변경하면 다른 DB로 쉽게 변경할 수 있다.
3. 1차 & 2차 캐시 지원
4. 트랜잭션 관리
5. HQL (Hibernate Query Language)
    1. Hibernate는 SQL이 아닌 **객체 기반의 쿼리 언어(HQL)**을 제공한다.

### 주요  구성 요소

1. SessionFactory (세션 팩토리)
    - **데이터베이스와의 연결 정보를 관리**하는 객체.
    - 애플리케이션이 시작될 때 한 번만 생성되고, `Session` 객체를 제공해.
    - **무겁기 때문에 애플리케이션에서 하나만 생성해야 함.**
2. Session (세션)
    - **DB와의 인터랙션(쿼리 실행, 데이터 저장, 수정, 삭제 등)을 담당**하는 객체.
    - `SessionFactory`에서 생성하며, 하나의 트랜잭션 범위 내에서 사용됨.
    - Hibernate의 1차 캐시를 담당함.
3. Transaction (트랜잭션)
    - 데이터의 **일관성을 유지하기 위해 트랜잭션을 관리하는 객체.**
    - `session.beginTransaction()`, `commit()`, `rollback()`을 사용하여 DB 작업을 제어함.
4. Query (쿼리)
    - Hibernate에서 데이터를 조회할 때 사용하는 객체.
    - `HQL`(Hibernate Query Language) 또는 `Criteria API`를 사용해서 데이터를 조회할 수 있음.

### 사용 예시

- hibernate.cfg.xml
    
    ```xml
    <hibernate-configuration>
        <session-factory>
            <!-- DB 연결 설정 -->
            <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
            <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/test_db</property>
            <property name="hibernate.connection.username">root</property>
            <property name="hibernate.connection.password">1234</property>
    
            <!-- Hibernate 설정 -->
            <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
            <property name="hibernate.show_sql">true</property>
            <property name="hibernate.hbm2ddl.auto">update</property>
        </session-factory>
    </hibernate-configuration>
    
    ```
    
- SessionFactory 설정
    
    ```java
    import org.hibernate.SessionFactory;
    import org.hibernate.cfg.Configuration;
    
    public class HibernateUtil {
        private static final SessionFactory sessionFactory = buildSessionFactory();
    
        private static SessionFactory buildSessionFactory() {
            try {
                return new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();
            } catch (Throwable ex) {
                throw new ExceptionInInitializerError(ex);
            }
        }
    
        public static SessionFactory getSessionFactory() {
            return sessionFactory;
        }
    
        public static void shutdown() {
            getSessionFactory().close();
        }
    }
    
    ```
    
- Hibernate로 데이터 저장
    
    ```java
    import org.hibernate.Session;
    import org.hibernate.Transaction;
    
    public class HibernateMain {
        public static void main(String[] args) {
            // Hibernate Session 열기
            Session session = HibernateUtil.getSessionFactory().openSession();
            Transaction transaction = null;
    
            try {
                // 트랜잭션 시작
                transaction = session.beginTransaction();
    
                // 객체 저장
                User user = new User("홍길동", "hong@example.com");
                session.save(user);
    
                // 트랜잭션 커밋
                transaction.commit();
                System.out.println("User 저장 완료!");
    
            } catch (Exception e) {
                if (transaction != null) {
                    transaction.rollback();
                }
                e.printStackTrace();
            } finally {
                // 세션 닫기
                session.close();
            }
    
            // Hibernate 종료
            HibernateUtil.shutdown();
        }
    }
    
    ```
    

### JPA & Hibernate

**JPA는 표준 API이고 Hibernate는 구현체**

| **비교 항목** | JPA (Java Persistence API) | Hibernate |
| --- | --- | --- |
| 정의 | Java 진영에서 만든 ORM 표준 명세 | JPA의 구현체 중 하나 (가장 많이 사용됨) |
| 역할 | ORM을 위한 인터페이스 제공 (구현 X) | JPA의 기능을 실제로 구현 |
| **SQL 자동 생성** | 지원 (하지만 직접 SQL 실행은 불가능) | 지원 + 더 많은 기능 제공 (예: `@Filter`, `@LazyCollection`) |
| **캐시 기능** | 1차 캐시 제공 | 1차 + 2차 캐시 제공 |
| **쿼리 언어** | JPQL 사용 | JPQL + HQL(Hibernate Query Language) |
| **스프링에서 주로 사용하는 방식** | `Spring Data JPA`와 함께 사용 | `Spring Data JPA + Hibernate` 조합으로 사용 |

## JPA

- 원래 Java에서는 **EJB(Entity Bean)**를 사용했지만, 너무 복잡했다.
- 이를 개선하기 위해 Gavin King이 **Hibernate**라는 오픈소스를 만들었다.
- 이후 Hibernate가 JPA의 표준이 되면서, 현재 **Spring에서는 JPA + Hibernate 조합**을 가장 많이 사용한다.
- JPA 자체는 **인터페이스(스펙)만 제공**하고, 실제 기능은 Hibernate, EclipseLink, OpenJPA 같은 **JPA 구현체**가 담당한다.

### JPA의 핵심 개념

1. Entity
    - **데이터베이스 테이블과 매핑되는 클래스**
    - `@Entity`를 사용하여 정의
2.  EntityManager (엔티티 매니저)
    - **JPA에서 데이터를 저장하고 관리하는 핵심 객체**
    - Hibernate의 `Session`과 유사한 역할 수행
3. Persistence Context (영속성 컨텍스트)
    - **JPA에서 엔티티 객체를 관리하는 공간**
    - `EntityManager`가 관리하며, 데이터베이스와의 동기화를 자동 처
4. JPQL (Java Persistence Query Language)
    - SQL과 비슷하지만 **엔티티 객체를 대상으로 쿼리하는 JPA 전용 언어**
    - `SELECT u FROM User u WHERE u.name = '홍길동'`

### 사용 예시 (Spring 없이 사용)

- persistence.xml
    
    ```java
    <persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
                 version="2.1">
        <persistence-unit name="jpa-example">
            <properties>
                <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/test_db"/>
                <property name="javax.persistence.jdbc.user" value="root"/>
                <property name="javax.persistence.jdbc.password" value="1234"/>
    
                <!-- Hibernate 설정 -->
                <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>
                <property name="hibernate.show_sql" value="true"/>
                <property name="hibernate.hbm2ddl.auto" value="update"/>
            </properties>
        </persistence-unit>
    </persistence>
    
    ```
    
- 엔티티(Entity) 클래스
    
    ```java
    import jakarta.persistence.*;
    
    @Entity
    @Table(name = "users")
    public class User {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        @Column(nullable = false)
        private String name;
    
        @Column(unique = true)
        private String email;
    
        public User() {}
    
        public User(String name, String email) {
            this.name = name;
            this.email = email;
        }
    
        // Getter & Setter
    }
    
    ```
    
- JPA를 활용한 데이터 저장
    
    ```java
    import jakarta.persistence.*;
    
    public class JpaMain {
        public static void main(String[] args) {
            // EntityManagerFactory 생성
            EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa-example");
            EntityManager em = emf.createEntityManager();
    
            // 트랜잭션 시작
            EntityTransaction tx = em.getTransaction();
            tx.begin();
    
            try {
                // 객체 저장
                User user = new User("홍길동", "hong@example.com");
                em.persist(user);
    
                // 트랜잭션 커밋
                tx.commit();
                System.out.println("User 저장 완료!");
    
            } catch (Exception e) {
                tx.rollback();
                e.printStackTrace();
            } finally {
                // EntityManager 닫기
                em.close();
                emf.close();
            }
        }
    }
    
    ```
    

### Spring Boot에서 JPA 사용

Spring Boot에서는 Hibernate 설정 없이 **Spring Data JPA**를 사용하면 간편하게 JPA 기능을 활용할 수 있다.

- application.properties
    
    ```java
    spring.datasource.url=jdbc:mysql://localhost:3306/test_db
    spring.datasource.username=root
    spring.datasource.password=1234
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    
    spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true
    ```
    
- 엔티티(Entity) 클래스
    
    ```java
    import jakarta.persistence.*;
    
    @Entity
    public class User {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String name;
        private String email;
    
        // 기본 생성자 및 Getter & Setter
    }
    
    ```
    
- JPA Repository 생성
    
    ```java
    import org.springframework.data.jpa.repository.JpaRepository;
    
    public interface UserRepository extends JpaRepository<User, Long> {
    }
    ```
    
- Service & Controller에서 JPA 활용
    
    ```java
    import org.springframework.stereotype.Service;
    import java.util.List;
    
    @Service
    public class UserService {
        private final UserRepository userRepository;
    
        public UserService(UserRepository userRepository) {
            this.userRepository = userRepository;
        }
    
        public void saveUser(String name, String email) {
            User user = new User();
            user.setName(name);
            user.setEmail(email);
            userRepository.save(user);
        }
    
        public List<User> getAllUsers() {
            return userRepository.findAll();
        }
    }
    
    ```