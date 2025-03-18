# Index

데이터를 효율적으로 검색하기 위해 테이블 전체를 조회하여 탐색하는 것이 아니라, 테이블의 특정 열(Column)들에 대해 조회할 수 있는 데이터 구조이다.

마치 책의 색인처럼, 필요한 데이터를 더 빠르게 찾을 수 있도록 돕는 역할을 한다.

<br>

## 1. Index의 기본 개념

- 검색 속도 향상
    - 테이블의 데이터를 순차적으로 탐색하지 않고도  원하는 데이터를 빠르게 찾을 수 있다.
- 추가 저장 공간 필요
    - 인덱스를  저장하기 위해 추가적인 디스크 공간이 필요
- 데이터 수정 시 성능 영향
    - 데이터 삽입, 삭제, 업데이트 작업 시 Index도 갱신되어야 하기  때문에 성능에 영향을 줄 수 있다.

<br>

### Index 생성 예제

```sql
-- 단일 열 인덱스 생성
CREATE INDEX idx_user_name ON users(name);

-- 복합 인덱스 생성
CREATE INDEX idx_user_name_email ON users(name, email);

-- 고유 인덱스 생성
CREATE UNIQUE INDEX idx_unique_email ON users(email);

-- 기본 키 인덱스는 테이블 생성 시 자동으로 생성됨
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

```

<br>

## 2. Index의 동작 방식

- **B-Tree 인덱스**: 대부분의 RDBMS에서 사용하는 기본 인덱스 형태로, 데이터를 정렬된 상태로 저장하고 탐색, 삽입, 삭제 시 효율적인 성능을 제공해.
- **Hash 인덱스**: 특정 값에 대한 정확한 매칭 검색에 유리하지만 범위 검색에는 적합하지 않아.
- **Bitmap 인덱스**: 열 값의 범위가 작거나 값이 중복되는 경우에 적합하며, 주로 데이터 웨어하우스에서 사용돼.
- **Full-text 인덱스**: 텍스트 검색에 최적화된 인덱스로, 긴 텍스트에서 키워드를 빠르게 검색할 때 사용돼.

<br>

## 3. **인덱스 사용의 장단점**

### **장점**

- 검색 속도가 빨라져 쿼리 성능이 향상됨.
- 정렬 및 그룹화 작업에 유리함.
- 중복 데이터 방지(고유 인덱스의 경우).

### **단점**

- 인덱스가 많아지면 삽입, 삭제, 업데이트 시 성능이 저하될 수 있음.
- 추가 저장 공간이 필요함.
- 너무 많은 인덱스를 사용하면 오히려 성능 저하를 초래할 수 있음.

<br>

## 4. Sequence와의 차이점

| 특징 | 인덱스 (Index) | 시퀀스 (Sequence) |
| --- | --- | --- |
| 역할 | 데이터를 빠르게 검색할 수 있도록 최적화 | 고유하고 순차적인 숫자 값을 생성 |
| 사용 대상 | 테이블의 특정 열  (Column) | 테이블과 독립적으로 동작 |
| 속도에 미치는 영향 | 검색 속도를 빠르게 함 | 직접적인 검색 성능에는 영향 없음 |
| 관리 방식 | 테이블과 연관되어 관리됨 | 독립적인 객체로 관리됨 |
| 생성 목적 | 빠른 검색, 정렬, 데이터 조회 | 고유 값 생성 (주로 기본 키나 트랜잭션 번호) |


<br>

### Index와 Sequence를 같이 사용하는 경우

시퀀스로 생성한 값은 일반적으로 `기본 키 (PK)`로 사용되고, 이 기본 키에 **인덱스가 자동으로 생성**돼. 즉, 시퀀스는 값을 생성하고, 인덱스는 그 값을 더 빠르게 조회하는 데 사용되는 거지.

```sql
-- 시퀀스 생성
CREATE SEQUENCE seq_order_id START WITH 1 INCREMENT BY 1;

-- 테이블 생성
CREATE TABLE orders (
    id NUMBER PRIMARY KEY,  -- 기본 키에 자동으로 인덱스 생성
    product_name VARCHAR2(50)
);

-- 시퀀스를 사용해 값 삽입
INSERT INTO orders (id, product_name) VALUES (seq_order_id.NEXTVAL, 'Laptop');
```

<br>

## 5. Index 사용 사례 예시

### 쇼핑몰 상품 테이블

- **상품명을 검색하거나 카테고리별로 상품을 조회**해야 하는 상황을 가정하자. 테이블에는 다음과 같은 구조가 있을 거야

```sql
CREATE TABLE products (
    id INT PRIMARY KEY,            -- 고유 상품 ID (기본 키, 자동으로 인덱스 생성)
    name VARCHAR(255) NOT NULL,    -- 상품 이름
    category VARCHAR(50),          -- 카테고리 이름
    price DECIMAL(10, 2),          -- 가격
    stock INT                      -- 재고 수량
);
```

<br>

### **문제점**

- 테이블에 **수백만 개의 상품 데이터**가 저장되어 있다고 가정하면, 특정 상품 이름이나 카테고리를 검색하는 데 시간이 오래 걸릴 수 있어.
- 이때, **`name`과 `category` 컬럼에 인덱스를 생성하면 검색 속도가 크게 향상**될 수 있어.

1. 단일 열 인덱스
    
    ```sql
    -- 상품 이름 컬럼에 인덱스 생성
    CREATE INDEX idx_product_name ON products (name);
    ```
    
    - 인덱스를 생성한 이후 "Laptop"이라는 이름을 가진 상품을 검색하려고 할 때
    
    ```sql
    SELECT * FROM products WHERE name = 'Laptop';
    ```
    
    - `idx_product_name` 인덱스를 통해 데이터베이스는 전체 테이블을 검색하지 않고 **빠르게 해당 데이터 위치를 찾아** 조회할 수 있다.
2. 복합 인덱스
    - 카테고리와 가격 범위로 상품 조회
    
    ```sql
    -- 복합 인덱스 생성 (카테고리 + 가격)
    CREATE INDEX idx_category_price ON products (category, price);
    ```
    
    - 'Electronics’ 라는 카테고리에서 ‘price’ 조건을 적용하려고 할 때
    
    ```sql
    SELECT * 
    FROM products 
    WHERE category = 'Electronics' AND price BETWEEN 500 AND 1500;
    ```
    
    - 복합 인덱스를 사용하면, 데이터베이스는 `category`로 먼저 필터링한 후 `price` 조건을 적용하여 훨씬 빠르게 결과를 반환할 수 있다.
