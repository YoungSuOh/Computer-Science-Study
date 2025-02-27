## Array

- 정적으로 필요한 만큼만 원소를 저장할 수 있는 공간이 할당
    - 이때 각 원소의 주소는 연속적으로 할당됨
- index를 통해 O(1)에 접근이 가능함
- 삽입 및 삭제는 O(N)
- 지정된 개수가 초과되면? → **배열의 크기를 재 할당한 후 복사**

## List

- 노드(Node)들의 연결로 이루어짐
- 크기 제한이 없음(heap 용량만 충분하다면)
- 다음 노드에 대한 참조를 통해 접근(O(N))
- 삽입과 삭제가 편함 O(1)

## ArrayList

- 동적으로 크기가 조정되는 배열
- 배열이 가득차면? → 알아서 그 크기를 2배로 할당하고 복사 수행
- 재할당에 걸리는 시간은 O(N)이지만, 자주 일어나는 일이 아니므로 접근 시간은 O(1)

## Stack (스택)

- LIFO 방식 (나중에 들어온 게 먼저 나감)
- 원소의 삽입 및 삭제가 top에서만 이루어짐
- 함수 호출 시 지역 변수, 매개 변수 정보를 저장하기 위한 공간을 스택으로 사용함

## Queue & Deque

- **큐(Queue)**:
    - **삽입(Enqueue): rear(뒤)에서 발생**
    - **삭제(Dequeue): front(앞)에서 발생**
    - FIFO 구조
- **덱(Deque, Double-Ended Queue)**:
    - **양쪽(front, rear) 모두에서 삽입과 삭제 가능**
    - 따라서 **스택(Stack)과 큐(Queue)의 성질을 모두 가짐**

FIFO 운영체제, 은행 대기열 등에 해당

### Priority Queue

FIFO 방식이 아닌 데이터를 근거로 한 우선순위를 판단하고, 우선순위가 높은 것부터 나감

**구현 방법**

1. 배열
    - 간단하게 구현이 가능
    - 데이터 삽입 및 삭제 과정을 진행 시, O(N)으로 비효율 발생 (한 칸씩 당기거나 밀어야 하기 때문)
        - 삽입 위치를 찾기 위해 배열의 모든 데이터를 탐색해야 함
        - Worst Cast : 우선 순위가 가장 낮을 경우
2. Linked List
    - 삽입 및 삭제 O(1)
    - 하지만 삽입 위치를 찾을 때는 배열과 마찬가지로 비효율 발
3. 힙
    - 힙은 위 2가지를 모두 효율적으로 처리가 가능함 → 따라서 우선 순위 큐는 대부분 힙으로 구현)
    - index 0은 안씀 → 부모와 자식을 2배 관계를 유지하기 위해사
    - 삽입 후, heapify 과정을 통해 힙의 모든 부모-자식 노드의 우선순위에 맞게 설정됨
    - 삭제는 root node를 삭제 후, 마지막 leaf node를 root node로 옮긴 뒤 heapify

## Tree

- 사이클이 없는 무방향 그래프
- 완전 이진트리 기준 높이는 logN
- 트리 순회 방법
    - 전위 순회, 중위 순회, 후위 순회

### BST

- 노드의 왼쪽은 노드의 값보다 작은 값들, 오른쪽 노드의 값보다 큰 값으로 구성
- 삽입 및 삭제, 탐색까지 이상적일 때는 모두 O(logN) 가능
- 만약 편향된 트리면 O(N)으로 최악의 경우가 발생

## Hash Table

- 효율적 탐색을 위한 자료구조
- key - value 쌍으로 이루어짐
- 해시 함수를 통해 입력받은 key를 정수값(index)로 대응시킴
- 충돌(collision)에 대한 고려 필요

### 충돌(collision) 해결방안

해시 테이블에서 중복된 값에 대한 충돌 가능성이 있기 때문에 해결방안을 세워야 함

### 1.선형 조사법(linear probing)

충돌이 일어난 항목을 해시 테이블의 다른 위치에 저장

```
예시)
ht[k], ht[k+1], ht[k+2] ...

※ 삽입 상황
충돌이 ht[k]에서 일어났다면, ht[k+1]이 비어있는지 조사함. 차있으면 ht[k+2] 조사 ...
테이블 끝까지 도달하면 다시 처음으로 돌아옴. 시작 위치로 돌아온 경우는 테이블이 모두 가득 찬 경우임

※ 검색 상황
ht[k]에 있는 키가 다른 값이면, ht[k+1]에 같은 키가 있는지 조사함.
비어있는 공간이 나오거나, 검색을 시작한 위치로 돌아오면 찾는 키가 없는 경우

```

### 2.이차 조사법

선형 조사법에서 발생하는 **집적화 문제를 완화**시켜 줌

```
h(k), h(k)+1, h(k)+4, h(k)+9 ...
```

### 3.이중 해시법

재해싱(rehasing)이라고도 함

충돌로 인해 비어있는 버킷을 찾을 때 추가적인 해시 함수 h'()를 사용하는 방식

```
h'(k) = C - (k mod C)

조사 위치
h(k), h(k)+h'(k), h(k) + 2h'(k) ...

```

### 4.체이닝

각 버킷을 고정된 개수의 슬롯 대신, 유동적 크기를 갖는 **연결리스트로 구성**하는 방식

충돌 뿐만 아니라 오버플로우 문제도 해결 가능

버킷 내에서 항목을 찾을 때는 연결리스트 순차 탐색 활용

### 5.해싱 성능 분석

```
a = n / M

a = 적재 비율
n = 저장되는 항목 개수
M = 해시테이블 크기

```

### 맵(map)과 해시맵(hashMap)의 차이는?

map 컨테이너는 이진탐색트리(BST)를 사용하다가 최근에 레드블랙트리를 사용하는 중

key 값을 이용해 트리를 탐색하는 방식임 → 따라서 데이터 접근, 삽입, 삭제는 O( logN )

반면 해시맵은 해시함수를 활용해 O(1)에 접근 가능

하지만 C++에서는 해시맵을 STL로 지원해주지 않는데, 충돌 해결에 있어서 안정적인 방법이 아니기 때문 (해시 함수는 collision 정책에 따라 성능차이가 큼)