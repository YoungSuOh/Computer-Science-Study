# Stored Procedure

데이터베이스에 저장된 SQL 쿼리들의 집합으로, 마치 하나의 함수처럼 실행하기 위한 쿼리의 집합

데이터베이스에서 SQL을 통해 작업을 하다 보면, 하나의 쿼리문으로 원하는 결과를 얻을 수 없을 때가 생긴다. 

원하는 결과물을 얻기 위해 사용할 여러줄의 쿼리문을 한 번의 요청으로 실행하면 좋지 않을까? 

또한, 인자 값만 상황에 따라 바뀌고 동일한 로직의 복잡한 쿼리문을 필요할 때마다 작성한다면 비효율적이지 않을까?

이럴 때 사용할 수 있는 것이 바로 `Stored Procedure` 이다.



## Stored Procedure의 구조

```sql
DELIMITER $$

CREATE PROCEDURE 프로시저명 (매개변수)
BEGIN
    -- SQL Statements
END $$

DELIMITER ;
```

- **DELIMITER**: 프로시저 정의가 끝나는 구문을 구분하기 위해 사용.
- **매개변수**: IN, OUT, INOUT 같은 매개변수 지정.
    - `IN`: 호출 시 값을 전달만.
    - `OUT`: 프로시저 내에서 값을 설정해 반환.
    - `INOUT`: 값을 전달받고 수정된 값을 반환.

## 사용 사례

1. 반복적인 조회 작업
    1. 만약 특정 고객 데이터를 자주 조회한다고 가정해보자.
        
        ```sql
        DELIMITER $$
        
        CREATE PROCEDURE GetCustomerInfo(IN customerId INT)
        BEGIN
            SELECT * 
            FROM Customers 
            WHERE id = customerId;
        END $$
        
        DELIMITER ;
        ```
        
    2. 호출
        
        ```sql
        CALL GetCustomerInfo(5);
        ```
        

1. 데이터 삽입 및 로그 기록
    1. 새로운 주문 데이터를 삽입하고, 동시에 로그를 남기는 작업
        
        ```sql
        DELIMITER $$
        
        CREATE PROCEDURE InsertOrder(IN customerId INT, IN productId INT, IN quantity INT)
        BEGIN
            -- 주문 데이터 삽입
            INSERT INTO Orders(customer_id, product_id, quantity, order_date)
            VALUES (customerId, productId, quantity, NOW());
        
            -- 로그 남기기
            INSERT INTO Logs(action, log_time)
            VALUES ('New order added', NOW());
        END $$
        
        DELIMITER ;
        
        ```
        
    2. 호출
        
        ```sql
        CALL InsertOrder(1, 101, 3);
        ```
        

## Stored Procedure의 장단점

### 장점

- **트래픽 감소**
    - 클라이언트가 직접 SQL문을 작성하지 않고, 프로시저명에 매개변수만 담아 전달하면 된다.
    - 즉, SQL문이 서버에 이미 저장되어 있기 때문에 클라이언트와 서버 간 네트워크 상 트래픽이 감소된다.
- **재사용 가능 (유지 보수)**
    - 한 번 작성해두면 여러 애플리케이션이나 사용자가 재사용 가능.
- **성능 향상**
    - 프로시저의 최초 실행 시 최적화 상태로 컴파일이 되며, 그 이후 프로시저 캐시에 저장된다.
    - 만약 해당 프로세스가 여러번 사용될 때, 다시 컴파일 작업을 거치지 않고 캐시에서 가져오게 된다.
- **보안**
    - 프로시저 내에서 참조 중인 테이블의 접근을 막을 수 있다.

### 단점

- 호환성
    - 구문 규칙이 SQL / PSM 표준과의 호환성이 낮기 때문에 코드 자산으로의 재사용성이 나쁘다.
- 성능
    - 문자 또는 숫자 연산에서 프로그래밍 언어인 C나 Java보다 성능이 느리다.
- 디버깅
    - 에러가 발생했을 때, 어디서 잘못됐는지 디버깅하는 것이 힘들 수 있다.