# Synchronous/Asynchronous & Blocking/Non-blocking

동기/비동기는 우리가 많이 들을 수 있는 말이다.

Blocking과 Synchronous, 그리고 Non-blocking과 Asynchronous를 서로 같은 개념이라고 착각하기 쉽다. 각자 어떤 의미를 가지는지 간단하게 살펴 보자.

## ⏳ **Synchronous vs. Asynchronous**

- **Synchronous(동기)**: 작업이 **순차적으로 실행**됨.
    
    ```python
    def sync_task():
        print("첫 번째 작업 시작")
        time.sleep(2)  # 2초 대기
        print("첫 번째 작업 완료")
    
        print("두 번째 작업 시작")
        time.sleep(2)  # 2초 대기
        print("두 번째 작업 완료")
    
    sync_task()
    print("모든 작업 완료")
    ```
    
    - 작업이 순차적으로 진행됨
    - `첫 번째 작업`이 끝나야 `두 번째 작업`이 실행됨
    - 따라서 총 4초 후에 모든 작업이 완료됨
- **Asynchronous(비동기)**: **작업을 실행한 후 바로 다음 코드 실행** → 이후 응답이 오면 결과 처리
    
    ```python
    import asyncio
    
    async def async_task(name, delay):
        print(f"{name} 시작")
        await asyncio.sleep(delay)  # 비동기 대기
        print(f"{name} 완료")
    
    async def main():
        await asyncio.gather(
            async_task("첫 번째 작업", 2),
            async_task("두 번째 작업", 2)
        )
        print("모든 작업 완료")
    
    asyncio.run(main())
    ```
    
    - 두 개의 작업이 동시에 실행되지만, 각 작업이 끝나는데 2초씩 걸림
    - 하지만 전체 실행 시간은 2초 (병렬 처리)

### 🚀 **핵심 차이점**

- **Synchronous**: 한 작업이 끝나야 다음 작업 실행
- **Asynchronous**: 여러 작업이 **동시에 실행될 수 있음**

## 🏛 Blocking vs. Non-blocking

- **Blocking(블로킹)**: 작업이 완료될 때까지 **현재 실행 중인 코드가 멈춘다**.
    
    ```python
    import time
    
    def blocking_function():
        print("작업 시작")
        time.sleep(3)  # 3초 동안 대기 (Blocking)
        print("작업 완료")
    
    blocking_function()
    print("다음 작업 실행")  # "작업 완료" 이후 실행됨
    
    ```
    
    - `time.sleep(3)`이 실행되는 동안 프로그램이 **완전히 멈춤** (Blocking).
    - 이후 코드(`print("다음 작업 실행")`)는 **3초 뒤**에 실행됨.
- **Non-blocking(논블로킹)**: 작업이 완료되지 않아도 **즉시 다음 코드로 넘어간다**.
    
    ```python
    import threading
    
    def non_blocking_function():
        print("작업 시작")
        threading.Timer(3, lambda: print("작업 완료")).start()
    
    non_blocking_function()
    print("다음 작업 실행")  # 즉시 실행됨
    
    ```
    
    - `threading.Timer(3, ...)`를 사용해 3초 후 실행하도록 설정.
    - 하지만 함수가 끝날 때까지 **대기하지 않음** (Non-blocking)
    - `print("다음 작업 실행")`이 **즉시 실행됨**

### 🚀 **핵심 차이점**

- **Blocking**: 함수가 **작업을 끝낼 때까지 기다림**
- **Non-blocking**: 함수가 **작업이 끝날 때까지 기다리지 않음**

## 🎯 네트워크 프로그래밍에서 적용 예시

| 방식 | 설명 | 예 |
| --- | --- | --- |
| Blocking I/O | `recv()` 를 호출하면 데이터 수신 완료까지 프로그램 멈춤 | `socket.recv(1024)` |
| Non-blocking I/O | `recv()` 를 호출해도 즉시 리턴, 나중에 데이터 확인 가능 | `socket.setblocking(False)` |
| Synchronous I/O | 네트워크 요청을 순차적으로 처리 | 일반적인 HTTP 요창 |
| Asynchronous I/O | 여러 네트워크 요청을 동시에 처리 | `asyncio` 기반 서버 |

## **🔥 언제 어떤 방식을 써야 할까?**

1. Blocking + Synchronous
    1. 단순한 코드 작성, 직관적이지만 성능이 낮음
    2. CPU가 놀게 되므로 비효율적
    3. 소규모 애플리케이션에 적합
2. Blocking + Asynchronous
    1. 병렬적으로 동작하지만, 각 요청은 여전히 Blocking
    2. 성능이 제한적
3. Non-blocking + Synchronous
    1. 여러 작업을 처리할 수 있지만, 응답 대기 시간이 길어질 수 있음
    2. 이벤트 루프를 활용하는 경우 (ex: Node.js, Java NIO)
4. Non-blocking + Asynchronous
    1. 네트워크 서버, 대규모 시스템에서 **가장 성능이 뛰어남**
    2. Node.js, Python asyncio, Java NIO 등에서 사용