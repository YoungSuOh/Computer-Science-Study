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
