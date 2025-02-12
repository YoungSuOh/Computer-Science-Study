## 스택 (Stack)

![image.png](attachment:058b1194-54d5-4c53-ae9f-4e6b2b2dd200:image.png)

입력과 출력이 한 곳(방향)으로 제한

**LIFO (Last In First Out, 후입선출) : 가장 나중에 들어온 것이 가장 먼저 나옴**

### 언제 사용하나?

함수의 콜스택, 문자열 역순 출력, 연산자 후위표기법

데이터 넣음 : push()

데이터 최상위 값 뺌 : pop()

비어있는 지 확인 : isEmpty()

꽉차있는 지 확인 : isFull()

push와 pop할 때는 해당 위치를 알고 있어야 하므로 기억하고 있는 '스택 포인터(SP)'가 필요함

스택 포인터는 다음 값이 들어갈 위치를 가리키고 있음 (처음 기본값은 -1)

```cpp
private int sp = -1;
```

- **push**

```cpp
public void push(Object o) {
    if(isFull(o)) {
        return;
    }
    
    stack[++sp] = o;
}
```

- 스택 포인터가 최대 크기와 같으면 return
- 아니면 스택의 최상위 위치에 값을 넣음

- pop

```cpp
public Object pop() {
    
    if(isEmpty(sp)) {
        return null;
    }
    
    Object o = stack[sp--];
    return o;
    
}
```

- 스택 포인터가 0이 되면 null로 return;
- 아니면 스택의 최상위 위치 값을 꺼내옴

- **isEmpty**

```cpp
private boolean isEmpty(int cnt) {
    return sp == -1 ? true : false;
}
```

- 입력 값이 최초 값과 같다면 true, 아니면 false

- **isFull**

```cpp
private boolean isFull(int cnt) {
    return sp + 1 == MAX_SIZE ? true : false;
}
```

- 스택 포인터 값+1이 MAX_SIZE와 같으면 true, 아니면 false

## 큐 (Queue)

입력과 출력을 한 쪽 끝(front, rear)으로 제한

**FIFO (First In First Out, 선입선출) : 가장 먼저 들어온 것이 가장 먼저 나옴**

### 언제 사용하나?

버퍼, 마구 입력된 것을 처리하지 못하고 있는 상황, BFS

![image.png](attachment:99788c8d-749f-4fca-b7c0-61d161c55bce:image.png)

큐의 가장 첫 원소를 front, 끝 원소를 rear라고 부름

큐는 **들어올 때 rear로 들어오지만, 나올 때는 front부터 빠지는 특성**을 가짐

접근방법은 가장 첫 원소와 끝 원소로만 가능

데이터 넣음 : enQueue()

데이터 뺌 : deQueue()

비어있는 지 확인 : isEmpty()

꽉차있는 지 확인 : isFull()

데이터를 넣고 뺄 때 해당 값의 위치를 기억해야 함. (스택에서 스택 포인터와 같은 역할)

이 위치를 기억하고 있는 게 front와 rear

front : deQueue 할 위치 기억

rear : enQueue 할 위치 기억