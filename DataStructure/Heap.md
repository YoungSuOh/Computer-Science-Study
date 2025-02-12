# Heap

힙은, 우선순위 큐를 위해 만들어진 자료구조이다.

완전 이진 트리의 일종이다.

힙 트리는 중복된 값 허용한다 (이진 탐색 트리는 중복값 허용 x)

![image.png](attachment:b42b309c-e97a-4a8d-9b4c-fde9a2bf3275:image.png)

- **최대 힙(max heap)**
    - 부모 노드의 키 값이 자식 노드의 키 값보다 크거나 같은 완전 이진 트리

- **최소 힙(min heap)**
    - 부모 노드의 키 값이 자식 노드의 키 값보다 작거나 같은 완전 이진 트리

- 부모 노드와 자식 노드 관계
    
    ```java
    왼쪽 자식 index = (부모 index) * 2
    
    오른쪽 자식 index = (부모 index) * 2 + 1
    
    부모 index = (자식 index) / 2
    ```
    

**힙의 삽입 (UpHeap)**

1. 힙에 새로운 요소가 들어오면, 일단 새로운 노드를 힙의 마지막 노드에 삽입
2. 새로운 노드를 부모 노드들과 교환

```cpp
void insert_max_heap(int x) {
    
    maxHeap[++heapSize] = x; 
    // 힙 크기를 하나 증가하고, 마지막 노드에 x를 넣음
    
    for( int i = heapSize; i > 1; i /= 2) {
        
        // 마지막 노드가 자신의 부모 노드보다 크면 swap
        if(maxHeap[i/2] < maxHeap[i]) {
            swap(i/2, i);
        } else {
            break;
        }
        
    }
}
```

**힙의 삭제 (Down Heap)**

1. 최대 힙에서 최대값은 루트 노드이므로 루트 노드가 삭제됨 (최대 힙에서 삭제 연산은 최대값 요소를 삭제하는 것)
2. 삭제된 루트 노드에는 힙의 마지막 노드를 가져옴
3. 힙을 재구성

```cpp
int delete_max_heap() {
    
    if(heapSize == 0) // 배열이 비어있으면 리턴
        return 0;
    
    int item = maxHeap[1]; // 루트 노드의 값을 저장
    maxHeap[1] = maxHeap[heapSize]; // 마지막 노드 값을 루트로 이동
    maxHeap[heapSize--] = 0; // 힙 크기를 하나 줄이고 마지막 노드 0 초기화
    
    for(int i = 1; i*2 <= heapSize;) {
        
        // 마지막 노드가 왼쪽 노드와 오른쪽 노드보다 크면 끝
        if(maxHeap[i] > maxHeap[i*2] && maxHeap[i] > maxHeap[i*2+1]) {
            break;
        }
        
        // 왼쪽 노드가 더 큰 경우, swap
        else if (maxHeap[i*2] > maxHeap[i*2+1]) {
            swap(i, i*2);
            i = i*2;
        }
        
        // 오른쪽 노드가 더 큰 경우
        else {
            swap(i, i*2+1);
            i = i*2+1;
        }
    }
    
    return item;
    
}
```