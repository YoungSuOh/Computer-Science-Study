# Linked List

![image (22)](https://github.com/user-attachments/assets/0fa741a2-1f6d-46c0-b48d-048578a388aa)


연속적인 메모리 위치에 저장되지 않는 선형 데이터 구조 (포인터를 사용해서 연결한다)

각 노드는 **데이터 필드**와 **다음 노드에 대한 참조**를 포함하는 노드로 구성

## **왜 Linked List를 사용하나?**

- 배열은 비슷한 유형의 선형 데이터를 저장하는데 사용할 수 있지만 제한 사항이 있음
    - 배열의 크기가 고정되어 있어 미리 요소의 수에 대해 할당을 받아야 함
    - 새로운 요소를 삽입하는 것은 비용이 많이 듬 (공간을 만들고, 기존 요소 전부 이동)

장점

- 동적 크기
- 삽입/삭제 용이

단점

- 임의로 액세스를 허용할 수 없음. 즉, 첫 번째 노드부터 순차적으로 요소에 액세스 해야함 (이진 검색 수행 불가능)
- 포인터의 여분의 메모리 공간이 목록의 각 요소에 필요

## 구현

### 노드

```cpp
// A linked list node 
struct Node 
{ 
  int data; 
  struct Node *next; 
}; 
```

### 노드 추가

- 앞쪽에 노드 추가

```cpp
void push(struct Node** head_ref, int new_data){
    struct Node* new_node = (struct Node*) malloc(sizeof(struct Node));

    new_node->data = new_data;

    new_node->next = (*head_ref);

    (*head_ref) = new_node;
}
```

- 특정 노드 다음에 추가

```cpp
void insertAfter(struct Node* prev_node, int new_data){
    if (prev_node == NULL){
        printf("이전 노드가 NULL이 아니어야 합니다.");
        return;
    }

    struct Node* new_node = (struct Node*) malloc(sizeof(struct Node));

    new_node->data = new_data;
    new_node->next = prev_node->next;

    prev_node->next = new_node;
    
}
```

- 끝쪽에 노드 추가

```cpp
void append(struct Node** head_ref, int new_data){
    struct Node* new_node = (struct Node*)malloc(sizeof(struct Node));

    struct Node *last = *head_ref;

    new_node->data = new_data;

    new_node->next = NULL;

    if (*head_ref == NULL){
        *head_ref = new_node;
        return;
    }

    while(last->next != NULL){
        last = last->next;
    }

    last->next = new_node;
    return;

}
```

### **Single Linked List**


![image (23)](https://github.com/user-attachments/assets/81b8c302-4142-426d-826f-aa2272f0cd1b)

- **한 방향(→)으로만 연결**됨 (노드가 다음 노드만 가리킴)
- *첫 번째 노드(head)**는 리스트의 시작점
- *마지막 노드(third)**는 `nullptr` (즉, 리스트의 끝을 나타냄)
- 중간 노드 삽입/삭제 시 앞 노드를 찾아야 해서 **시간 복잡도 O(n) 필요**
- **단순하고 메모리 절약** (뒤로 가는 포인터가 없음)

📌 **사용 예시:**

- 기본적인 리스트 구조 (ex: **Stack**, **Queue**)
- 간단한 데이터 저장 (ex: **기본적인 데이터 연결**)

### Doubly Linked List

![image (24)](https://github.com/user-attachments/assets/7a72e338-9775-48e0-aee6-d8674ec2e2fc)


- **양방향(↔) 연결** 가능 (각 노드가 앞과 뒤를 가리킴)
- **이전 노드(prev) 포인터 추가**, 따라서 **뒤로 이동 가능**
- **삽입/삭제가 O(1)** (중간에 삽입할 때 앞 노드를 찾으면 빠름)
- 하지만 **메모리 사용량이 증가** (prev 포인터 추가됨)

📌 **사용 예시:**

- **뒤로 가기가 필요한 경우 (ex: 웹 브라우저 히스토리)**
- **LRU 캐시** (가장 오래된 데이터를 쉽게 삭제할 수 있음)

### Circular Singly Linked List

![image (25)](https://github.com/user-attachments/assets/bc8b451d-12c4-41ed-b188-1cae9fffce85)


- 마지막 노드가 `nullptr`이 아니라 **처음(head)으로 돌아감**
- **항상 순환** (끝이 없음)
- 노드 탐색 시 무한 루프 가능 → **종료 조건이 필요함**
- **Queue (원형 큐)** 같은 곳에서 사용

📌 **사용 예시:**

- **Round-robin 스케줄링** (CPU 스케줄링)
- **멀티플레이어 게임 턴 시스템**
- **음악 플레이어에서 반복 재생**

### Circular Doubly Linked List

![image (26)](https://github.com/user-attachments/assets/dd6cc496-09c9-4e24-85a2-5011af60f24a)


- 이중 연결 리스트처럼 **양방향 이동 가능(↔)**
- 마지막 노드가 첫 노드를 가리켜서 **끝이 없이 계속 순환**
- **메모리 사용량 증가 (prev + next 필요)**
- 특정 노드를 찾을 필요 없이 **항상 순환**하므로 **빠른 접근 가능**

📌 **사용 예시:**

- **시계 시스템 (ex: 운영체제 프로세스 스케줄링)**
- **데이터 버퍼** (계속해서 데이터가 들어오고 빠지는 경우)
- **게임 UI에서 메뉴 순환**
