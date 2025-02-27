# Transaction

## 트랜잭션이란?

> 데이터베이스의 상태를 변화시키기 위해 수행하는 작업 단위
> 
<br>
상태를 변화시킨다는 것 → **SQL 질의어를 통해 DB에 접근하는 것**

```
- SELECT
- INSERT
- DELETE
- UPDATE

```

### Select 문은 트랜잭션이 아니지 않나?

- 많은 사람이 트랜잭션이라고 하면 데이터 변경을 일으키는 INSERT, UPDATE, DELETE 같은 DML 구문만 생각하는 경우가 많다
- 하지만 실제로는 SELECT도 트랜잭션 isolation level 안에서 수행되는 작업으로 보기에, DBMS 입장에선 SELECT도 트랜잭션 범위에 들어간다.
    - 트랜잭션이 시작된 상태에서 SELECT 문을 실행하면, 해당 트랜잭션의 isolation level 설정 등에 의해 일관된 뷰(consistent view)를 읽게 된다.
    - 예를 들어 REPEATABLE READ나 SERIALIZABLE 같은 격리 수준에선, 한 트랜잭션 안에서 같은 SELECT 문을 여러 번 실행하더라도 동일한 결과를 보장해줘야 하기 때문에
<br>
작업 단위 → **많은 SQL 명령문들을 사람이 정하는 기준에 따라 정하는 것**

```
예시) 사용자 A가 사용자 B에게 만원을 송금한다.

* 이때 DB 작업
- 1. 사용자 A의 계좌에서 만원을 차감한다 : UPDATE 문을 사용해 사용자 A의 잔고를 변경
- 2. 사용자 B의 계좌에 만원을 추가한다 : UPDATE 문을 사용해 사용자 B의 잔고를 변경

현재 작업 단위 : 출금 UPDATE문 + 입금 UPDATE문
→ 이를 통틀어 하나의 트랜잭션이라고 한다.
- 위 두 쿼리문 모두 성공적으로 완료되어야만 "하나의 작업(트랜잭션)"이 완료되는 것이다. `Commit`
- 작업 단위에 속하는 쿼리 중 하나라도 실패하면 모든 쿼리문을 취소하고 이전 상태로 돌려놓아야한다. `Rollback`

```

**즉, 하나의 트랜잭션 설계를 잘 만드는 것이 데이터를 다룰 때 많은 이점을 가져다준다.**
<br><br>
### 트랜잭션 특징

---

- 원자성(Atomicity)
    
    > 트랜잭션이 DB에 모두 반영되거나, 혹은 전혀 반영되지 않아야 된다.
    > 
- 일관성(Consistency)
    
    > 트랜잭션의 작업 처리 결과는 항상 일관성 있어야 한다.
    > 
- 독립성(Isolation)
    
    > 둘 이상의 트랜잭션이 동시에 병행 실행되고 있을 때, 어떤 트랜잭션도 다른 트랜잭션 연산에 끼어들 수 없다.
    > 
- 지속성(Durability)
    
    > 트랜잭션이 성공적으로 완료되었으면, 결과는 영구적으로 반영되어야 한다.
    > 

### Commit

하나의 트랜잭션이 성공적으로 끝났고,  DB가 일관성있는 상태일 때 이를 알려주기 위해 사용하는 연산

### Rollback

하나의 트랜잭션 처리가 비정상적으로 종료되어 트랜잭션 원자성이 깨진 경우

transaction이 정상적으로 종료되지 않았을 때, last consistent state (예) Transaction의 시작 상태) 로 roll back 할 수 있음.

*상황이 주어지면 DB 측면에서 어떻게 해결할 수 있을지 대답할 수 있어야 함*

### DBMS의 구조
![image (5)](https://github.com/user-attachments/assets/1027035a-3f1c-42df-9ac6-f61a35bd0eb1)


1. 질의 처리기
2. 저장시스템
3. 시스템 카탈로그
    1. 테이블, 인덱스, 뷰, 사용자 계정 등 DB 객체들에 대한 정보를 메타데이터로 관리하는 공간
4. 저장 데이터 : 데이터베이스가 관리하는 **실제 데이터를 담는 물리적 저장소**
