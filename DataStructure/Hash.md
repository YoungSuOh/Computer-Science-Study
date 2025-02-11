# Hash

![image.png](attachment:fdeb9560-b7db-4870-bc51-df43b672fe1f:image.png)

해시(Hash) 자료구조는 **키(key)와 값(value)을 매핑**하는 방식으로 데이터를 저장하는 구조이다.

빠른 검색, 삽입, 삭제 연산이 가능해서 컴퓨터 과학에서 많이 사용된다.

대표적인 예로 **해시 맵(Hash Map), 해시 테이블(Hash Table), 딕셔너리(Dictionary)** 등이 있다.

## Hash 자료구조의 개념

해시는 **해시 함수(Hash Function)** 를 사용해 키를 특정한 인덱스로 변환한 후, 해당 인덱스에 값을 저장하는 구조이다.

- **해시 함수**: 키를 일정한 범위의 인덱스로 변환하는 함수
- **해시 테이블**: 해시 함수를 사용해 데이터를 저장하는 배열 기반의 자료구조
- **해시 충돌(Hash Collision)**: 서로 다른 키가 동일한 인덱스로 매핑될 때 발생

## 해시 함수 (Hash Function)

해시 함수는 입력받은 키를 고정된 크기의 숫자로 변환하는 함수이다.

좋은 해시 함수는 충돌을 최소화하고, 키를 균등하게 분산해야 한다.

### ✅ 좋은 해시 함수의 조건

- **빠르게 계산 가능**해야 한다.
- **고유한 값을 생성**해야 한다. (충돌 최소화)
- **균등한 분포**를 가져야 한다.
- **입력 값이 비슷해도 해시 값이 크게 차이 나야 한다.** (해싱 편향 방지)

### 해시 함수 예제

```java
public int simpleHash(String key, int tableSize) {
    int hash = 0;
    for (int i = 0; i < key.length(); i++) {
        hash = (hash * 31 + key.charAt(i)) % tableSize;  // 31은 소수(Prime Number) 사용
    }
    return hash;
}

```

- 위 함수는 문자열의 각 문자를 숫자로 변환하고, 31을 곱하면서 해시 값을 생성하는 방식이야.

## 해시 충돌(Hash Collision) 해결 방법

해시 충돌이 발생하면 **두 개 이상의 키가 같은 인덱스에 저장**되기 때문에 해결해야 한다.
해결 방법으로 **체이닝(Chaining)** 과 **오픈 어드레싱(Open Addressing)** 이 있다.

### ✅ 1) 체이닝(Chaining) (연결 리스트 사용)

같은 해시 값이 발생하면, **해당 인덱스에 연결 리스트(Linked List) 형태로 데이터를 저장**하는 방식이야.

### 🔹 장점

- 충돌이 발생해도 유연하게 저장할 수 있음.
- 테이블 크기가 부족해도 저장 가능.

### 🔹 단점

- **추가적인 메모리**(포인터 관리)가 필요.
- 탐색 시 연결 리스트를 따라가야 하므로 **속도가 느려질 수 있음**.

### 🔹 예제 (Java의 HashMap 내부 구조)

```java
class HashNode {
    String key;
    int value;
    HashNode next;

    public HashNode(String key, int value) {
        this.key = key;
        this.value = value;
        this.next = null;
    }
}

class HashTable {
    private HashNode[] table;
    private int size;

    public HashTable(int size) {
        this.size = size;
        table = new HashNode[size];
    }

    public void put(String key, int value) {
        int index = key.hashCode() % size;
        HashNode newNode = new HashNode(key, value);

        if (table[index] == null) {
            table[index] = newNode;
        } else {
            HashNode current = table[index];
            while (current.next != null) {
                current = current.next;
            }
            current.next = newNode;
        }
    }
}

```

### ✅ 2) 오픈 어드레싱(Open Addressing) (배열 내에서 해결)

충돌이 발생하면, 다른 빈 공간을 찾아서 저장하는 방식이다.

여러 가지 기법이 있는데 대표적으로 **선형 탐사(Linear Probing)**, **이차 탐사(Quadratic Probing)**, **이중 해싱(Double Hashing)** 이 있다.

- 선형 탐사 (Linear Probing)
    - 충돌이 발생하면 **다음 빈 공간**을 찾아서 저장하는 방식.
    - `index = (hash + i) % tableSize`
- 이차 탐사 (Quadratic Probing)
    - 충돌이 발생하면 **제곱 형태로 증가**하며 탐색.
    - `index = (hash + i^2) % tableSize`
- 이중 해싱 (Double Hashing)
    - 충돌이 발생하면 **두 번째 해시 함수**를 사용해서 새로운 위치를 찾음.
    - `index = (hash1 + i * hash2) % tableSize`

## 해시 자료구조의 시간 복잡도

| 연산 | 평균 시간 복잡도 | 최악 시간 복잡도 (충돌 많을 때) |
| --- | --- | --- |
| 검색 | O(1) | O(n) |
| 삽입 | O(1) | O(n) |
| 삭제 | O(1) | O(n) |
- **평균적으로 O(1)** 의 성능을 가지지만, 충돌이 많아지면 **최악의 경우 O(n)** 이 될 수 있어.
- 충돌이 많을 경우 성능이 떨어지므로 **좋은 해시 함수**와 **적절한 해시 테이블 크기 설정**이 중요해.

## 해시 자료구조의 활

### ✅ 1) 데이터 검색 및 저장

- **HashMap (Java), Dictionary (Python), HashTable (C++)** 등에서 사용
- 키-값 구조로 빠르게 데이터를 저장하고 검색할 때 활용

### ✅ 2) 캐싱(Cache)

- **LRU Cache (Least Recently Used)** 와 같은 알고리즘에서 사용
- 웹 브라우저, DB 쿼리 결과 등을 캐싱할 때 활용

### ✅ 3) 데이터 중복 제거

- **Set (HashSet)** 을 활용하면 중복 없이 데이터를 저장 가능
- 중복 체크가 필요할 때 사용 (ex. 방문한 웹페이지 기록)

### ✅ 4) 암호화 및 보안

- **해시 함수(SHA-256, MD5 등)** 를 이용해 비밀번호 암호화, 데이터 무결성 검사

### ✅ 5) 로드 밸런싱 및 분산 시스템

- **Consistent Hashing (일관된 해싱)** 을 활용해 서버 부하를 균등하게 분산

## 해시 버킷 동적 확장

해시 테이블의 크기가 너무 작으면 **해시 충돌이 자주 발생**해서 성능이 저하될 수 있어.

반대로 **처음부터 너무 크게 만들면 메모리 낭비**가 심해지겠지.

그래서 **적절한 순간에 해시 테이블의 크기를 동적으로 늘리는 방식**을 사용해.

이 기준이 되는 게 **Load Factor(부하율)** 이야.

### ✅ **Load Factor (부하율)**

- **공식:** `Load Factor = (저장된 데이터 개수) / (해시 버킷 개수)`
- 보통 `0.7 ~ 0.8` 을 넘으면 **버킷 크기를 확장**하는 게 일반적이야.

## 🔄 **리해싱(Rehashing)이란?**

기존 저장되어 있는 값들을 다시 해싱하여 새로운 키를 부여하는 것을 말한다

### 👉 **리해싱이 필요한 이유**

해시 테이블 크기를 늘리면 **기존 키들이 새로운 위치로 이동할 수 있어**.

왜냐하면 **해시 함수는 버킷 크기에 따라 인덱스를 계산**하는데, 버킷 크기가 바뀌면 해시 값도 달라질 수 있거든.

그래서 **모든 기존 데이터를 새 테이블에 다시 해싱하는 과정**을 **리해싱(Rehashing)** 이라고 해.

### 🔹 **리해싱 과정**

1. **해시 버킷 크기 증가**
    - 기존 크기의 **2배**로 확장하는 경우가 많음.
2. **기존 데이터들을 새 버킷에 다시 해싱**
    - 새로운 크기에 맞춰 **모든 키를 다시 해싱**해서 재배치
    - 기존 데이터가 500개라면 500개 모두 다시 해싱해야 한다.

### 🔹 **리해싱 과정 예제**

### 🔹 기존 상태 (버킷 크기 `5`)

| 키 | 해시 값 (`key % 5`) | 저장 위치 |
| --- | --- | --- |
| 7 | `7 % 5 = 2` | **2번** |
| 12 | `12 % 5 = 2` | **3번 (충돌 해결 후 저장)** |
| 17 | `17 % 5 = 2` | **4번 (충돌 해결 후 저장)** |

### 🔹 리해싱 후 (버킷 크기 `10`)

| 키 | 새 해시 값 (`key % 10`) | 새로운 저장 위치 |
| --- | --- | --- |
| 7 | `7 % 10 = 7` | **7번** |
| 12 | `12 % 10 = 2` | **2번** |
| 17 | `17 % 10 = 7` | **7번 (충돌 해결 후 저장)** |

👉 **버킷 크기가 달라지면 기존의 저장 위치를 유지할 수 없기 때문에, 모든 데이터를 다시 해싱해야 한다!**

따라서 너무 자주 리해싱이 발생하면 성능이 저하되기 때문에 보통 **load factor가 0.7~0.8 이상이 될 때만 리해싱을 수행하도록 설정하여 필요할 때만 리해싱을 수행하여 성능 저하는 방지한다.**

### 리해싱 예제

자바의 `HashMap` 은 `load factor` 가 `0.75` 를 넘으면 자동으로 **버킷 크기를 2배로 확장 & 리해싱**해

```java
import java.util.HashMap;

public class RehashingExample {
    public static void main(String[] args) {
        HashMap<Integer, String> map = new HashMap<>(4, 0.75f); // 초기 버킷 크기 4, load factor 0.75
        map.put(1, "A");
        map.put(2, "B");
        map.put(3, "C");
        map.put(4, "D"); // Load Factor 0.75에 도달 -> 자동 리해싱 발생!

        System.out.println(map); // 내부적으로 리해싱 후 재배치됨
    }
}

```

### **해시 충돌이 없다고 가정하면?**

- 초기 크기 `4` → 데이터 4개 저장 → **load factor 1.0 (초과됨!)**
- 자동으로 버킷 크기가 `8`로 증가
- 기존 데이터 4개를 **새로운 해시 값으로 다시 저장** (리해싱 발생)