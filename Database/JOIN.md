# JOIN

두 개 이상의 테이블이나 데이터베이스를 연결하여 데이터를 검색하는 방법

- EX_TABLE
    
    
    | NO_EMP | NAME | DEPT |
    | --- | --- | --- |
    | 1 | Alice | HR |
    | 2 | Bob | IT |
    | 3 | Charlie | Finance |
- JOIN_TABLE
    
    
    | NO_EMP | AGE | SALARY |
    | --- | --- | --- |
    | 1 | 25 | 50000 |
    | 2 | 30 | 60000 |
    | 4 | 40 | 70000 |
<br><br>


## INNER JOIN

![image (6)](https://github.com/user-attachments/assets/8666ae8c-caf2-49c0-b217-8715cdd453cf)


```sql
SELECT
A.NAME, B.AGE
FROM EX_TABLE A
INNER JOIN JOIN_TABLE B ON A.NO_EMP = B.NO_EMP
```

- 교집합으로, 기준 테이블과 join 테이블의 중복된 값을 보여준다.

쿼리 결과

| NAME | AGE |
| --- | --- |
| Alice | 25 |
| Bob | 30 |
- 보면 서로 NO_EMP가 다른 경우에는 데이터가 나오지 않은 것을 볼 수 있다.

<br><br>

## LEFT OUTER JOIN

![image (7)](https://github.com/user-attachments/assets/f131104c-25c2-4e89-843b-8b6f1b001ca3)


```sql
SELECT
A.NAME, B.AGE
FROM EX_TABLE A
LEFT OUTER JOIN JOIN_TABLE B ON A.NO_EMP = B.NO_EMP
```

- 왼쪽 테이블 기준으로 JOIN 한다고 생각하면 된다.

쿼리 결과

| NAME | AGE |
| --- | --- |
| Alice | 25 |
| Bob | 30 |
| Charlie | NULL |
- 보면 JOIN TABLE 에서는 Charlie에 대한 데이터는 없기 때문에 `NULL` 로 표시된다

<br><br>

## **RIGHT OUTER JOIN**

![image (8)](https://github.com/user-attachments/assets/1a70aa6e-7cc1-428a-a3f8-4003a944c4c4)


- LEFT OUTER JOIN과는 반대로 오른쪽 테이블 기준으로 JOIN하는 것이다

<br><br>

## **FULL OUTER JOIN**

![image (9)](https://github.com/user-attachments/assets/a28e1c11-3055-46b5-a9a4-5721a48606d3)


```sql
SELECT
A.NAME, B.AGE
FROM EX_TABLE A
FULL OUTER JOIN JOIN_TABLE B ON A.NO_EMP = B.NO_EMP
```

- 양쪽 테이블 모두 기준으로 하여, **일치하지 않는 값도 포함**. 일치하지 않는 값이 있는 경우 `NULL`로 채워짐.

쿼리 결과

| NAME | AGE |
| --- | --- |
| Alice | 25 |
| Bob | 30 |
| Charlie | NULL |
| NULL | 40 |
- 보면 일치하지 않는 값은 `NULL` 로 채워지는 것을 볼  수 있다.

<br><br>

## **CROSS JOIN**

![image (10)](https://github.com/user-attachments/assets/f3afd07d-00db-4050-9284-52c9e54b91e1)


```sql
SELECT
A.NAME, B.AGE
FROM EX_TABLE A
CROSS JOIN JOIN_TABLE B
```

- 카타시안 곱처럼 모든 경우의 수의 데이터들을 확인할 수 있다.

쿼리 결과

| NAME | AGE |
| --- | --- |
| Alice | 25 |
| Alice | 30 |
| Alice | 40 |
| Bob | 25 |
| Bob | 30 |
| Bob | 40 |
| Charlie | 25 |
| Charlie | 30 |
| Charlie | 40 |

### 사용 사례

- **모든 조합을 확인해야 할 때**
    - 예를 들어, 제품 테이블과 할인율 테이블을 조합해 각각의 할인율에 따른 가격 계산이 필요할 때.
 
<br><br>

## SELF JOIN

![image (11)](https://github.com/user-attachments/assets/6ac39c36-d534-4b3b-83f5-d07ac68899f3)


```sql
SELECT
A.NAME, B.AGE
FROM EX_TABLE A, EX_TABLE B
```

- 자기자신과 자기자신을 조인하는 것이다.
- 하나의 테이블을 여러번 복사해서 조인한다고 생각하면 편하다.

쿼리 결과

| A.NAME | B.NAME |
| --- | --- |
| Alice | Alice |
| Alice | Bob |
| Alice | Charlie |
| Bob | Alice |
| Bob | Bob |
| Bob | Charlie |
| Charlie | Alice |
| Charlie | Bob |
| Charlie | Charlie |

### 사용 사례

- **같은 테이블 내에서 계층 구조를 비교해야 할 때**
    - 직원 테이블에서 상사와 부하 직원 정보를 가져오는 경우.
- **테이블의 값 간 비교가 필요할 때**
    - 한 제품이 다른 제품보다 더 비싼지 비교하거나, 같은 조건의 데이터를 그룹핑하고자 할 때.
- **자기 자신과 특정 관계를 탐색할 때**
    - 조직 구조, 네트워크 그래프 등에서의 관계 탐색.

## JOIN을 사용한 실제 사례가 있나요?

네이버 클라우드 캠프에서 JSP 기반 프로젝트를 진행하며 대댓글 기능을 구현한 경험이 있습니다. 처음에는 모든 데이터를 조회한 후 JavaScript에서 트리 구조를 생성했지만, 데이터가 많아질수록 렌더링 성능이 저하되었습니다.

이를 해결하기 위해 데이터베이스에서 `SELF-JOIN`과 `WITH RECURSIVE`를 사용해 계층적으로 댓글을 조회하고, `INNER JOIN`을 통해 부모 댓글과 대댓글을 연결하여 불필요한 연산을 최소화했습니다.

최종적으로, 데이터베이스에서 계층 정렬을 수행한 후 JavaScript에서 `DFS`를 활용해 트리 구조를 렌더링하는 방식으로 변경하여 클라이언트 부하를 줄이고 전체적인 성능을 개선할 수 있었습니다.

### `INNER JOIN` 을 하면 왜 불필요한 연산이 최소화되는가?

- `NNER JOIN`은 두 테이블(혹은 서브쿼리 결과) 간의 **연결된 데이터**만을 가져오기 때문에, **연관된 레코드들**만 조회하며, 부모-자식 관계가 성립하지 않는 데이터는 제외한다.
- 따라서 INNER JOIN을 서브 쿼리에 넣었을 때, 재귀 쿼리에서 더 이상 내려가지 않게 되어 불필요한 연산을 최소화할 수 있다.
