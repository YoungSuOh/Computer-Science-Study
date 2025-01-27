# SQL Injection

해커에 의해 조작된 SQL 쿼리문이 데이터베이스에 그대로 전달되어 비정상적 명령을 실행하는 공격 기법

## 공격 방법

1. 인증 우회
    - 보통 로그인을 할 때, 아이디와 비밀번호를 input 창에 입력하게 된다. 쉽게 이해하기 위해 가벼운 예를 들어보자.
    - 아이디가 abc, 비밀번호가 만약 1234일 때 쿼리는 아래와 같은 방식으로 전송될 것이다.
        
        ```sql
        SELECT * FROM USER WHERE ID = "abc" AND PASSWORD = "1234";
        ```
        
    - SQL Injection으로 공격할 때, input 창에 비밀번호를 입력함과 동시에 다른 쿼리문을 함께 입력하는 것이다.
        
        ```sql
        SELECT * FROM USER WHERE ID = "abc" AND PASSWORD = "1234" OR 1=1;
        ```
        
    - 보안이 완벽하지 않은 경우, 이처럼 기본 쿼리문의 WHERE 절에 OR문을 추가하여 `'1' = '1'`과 같은 true문을 작성하여 무조건 적용되도록 수정한 뒤 DB를 마음대로 조작할 수도 있다
2. 데이터 노출
    - 시스템에서 발생하는 에러 메시지를 이용해 공격하는 방법이다.
    - 보통 에러는 개발자가 버그를 수정하는 면에서 도움을 받을 수 있는 존재다. 해커들은 이를 역이용해 악의적인 구문을 삽입하여 에러를 유발시킨다.
    - 즉 예를 들면, 해커는 **GET 방식으로 동작하는 URL 쿼리 스트링을 추가하여 에러를 발생**시킨다. 이에 해당하는 오류가 발생하면, 이를 통해 해당 웹앱의 데이터베이스 구조를 유추할 수 있고 해킹에 활용한다.

## 방어 방법

1. **input 값을 받을 때, 화이트리스트 방식(허용된 패턴만 입력 가능)을 적용**
    
    ```java
    String idPattern = "^[a-zA-Z0-9]{4,20}$";  // 알파벳, 숫자 4~20자만 허용
    if (!input.matches(idPattern)) {
        throw new IllegalArgumentException("Invalid input");
    }
    ```
    
2.   **권한 분리와 최소 권한 원칙 추가**
    - 뷰(View) 활용하여 데이터베이스 계정은 **데이터 읽기(read) 권한만** 부여하고, 테이블 수정 권한은 차단
3.  **preparestatement,** Hibernate, JPA 같은 **ORM(Object Relational Mapping)** 도구를 사용
    - ORM은 기본적으로 SQL 바인딩과 escaping을 처리해 주기 때문
    - JDBC Preparestatement
        - 문제 상황
            
            ```java
            String username = "abc";
            String password = "1234 OR 1=1";
            
            String query = "SELECT * FROM USERS WHERE username = '" + username + "' AND password = '" + password + "'";
            
            Statement stmt = connection.createStatement();
            ResultSet rs = stmt.executeQuery(query);
            
            ```
            
            - 실행 되는 쿼리문
                
                ```java
                SELECT * FROM USERS WHERE username = 'abc' AND password = '1234' OR 1=1;
                ```
                
        - PreparedStatement로 해결
            
            ```java
            String query = "SELECT * FROM USERS WHERE username = ? AND password = ?";
            
            try (PreparedStatement pstmt = connection.prepareStatement(query)) {
                String username = "abc";
                String password = "1234' OR '1'='1";
            
                // 같은 PreparedStatement 코드
                pstmt.setString(1, username); 
                pstmt.setString(2, password);
            
                // 쿼리 실행
                ResultSet rs = pstmt.executeQuery();
            
                // 결과 처리
                while (rs.next()) {
                    System.out.println("User ID: " + rs.getInt("id"));
                    System.out.println("Username: " + rs.getString("username"));
                }
            }
            ```
            
            - 실행 되는 쿼리문
                
                ```java
                SELECT * FROM USERS WHERE username = 'abc' AND password = '1234'' OR ''1''=''1';
                
                ```
                
                - ⭐⭐ 특수문자 `'`는 SQL 문법에서의 의미를 잃고, 데이터로 escape 처리된다
    - JPA (Java Persistence API) - Hibernate
        
        ```java
        public User findUserByUsername(String username) {
            String jpql = "SELECT u FROM User u WHERE u.username = :username";
            return entityManager.createQuery(jpql, User.class)
                                .setParameter("username", username)
                                .getSingleResult();
        }
        ```
        
        - 여기서 `setParameter`를 사용하면 `username` 값이 자동으로 escaping 처리돼, 특수문자가 포함되어도 악의적인 SQL 구문으로 실행되지 않아.
    - Spring Data JPA
        
        ```java
        @Repository
        public interface UserRepository extends JpaRepository<User, Long> {
            User findByUsername(String username);
        }
        ```
        
        - `findByUsername` 메서드는 Spring Data JPA가 쿼리를 자동으로 생성해 실행함.
        - 이 경우에도 사용자 입력값은 **Prepared Statement**로 처리되기 때문에 SQL Injection을 방지할 수 있어.