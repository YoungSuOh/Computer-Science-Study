# CPU Scheduling

## 스케줄링

> CPU를 잘 사용하기 위한 프로세스를 잘 배정하기
> 

- 조건 : 오버헤드 ↓ / 사용률 ↑ / 기아 현상 ↓

- 컴퓨터 시스템의 운영 방식에 따라 구분된 운영체제 유형

| **구분** | **Batch System** | **Interactive System** | **Real-time System** |
| --- | --- | --- | --- |
| **작업 처리 방식** | 일괄 처리 | 실시간 사용자 요청 처리 | 실시간 작업 처리 |
| **반응 시간** | 대기 시간이 길다 | 사용자 요청에 즉각 반응 | 엄격히 제한된 시간 내 반응 |
| **상호작용** | 없음 | 사용자와 실시간 상호작용 | 작업마다 시간 엄수 필요 |
| **주요 용도** | 대량 데이터 처리(처리량이 중요) | 사용자 중심의 작업 | 안전, 제어, 실시간 시스템 |
| **예시** | 급여 처리, 데이터 배치 작업 | 웹 브라우저, 터미널 | 항공기 제어, 공장 자동화 |

## 선점 / 비선점 스케줄링

- 선점: OS가 CPU의 사용권을 선점할 수 있는 경우, 강제 회수하는 경우 (처리시간 예측 어려움)
- 비선점 : 프로세스 종료 or I/O 등의 이벤트가 있을 때까지 실행 보장 (처리시간 예측 용이함)

### 프로세스 상태

![image](https://github.com/user-attachments/assets/75c1ad4a-1f5d-4e68-bc24-e2a44324586a)

1. **승인 (Admitted)** 
    1. 프로세스 생성이 가능하여 승인됨.
2. **스케줄러 디스패치 (Scheduler Dispatch)** 
    1. 준비 상태에 있는 프로세스 중 하나를 선택하여 실행시키는 것.
3. **인터럽트 (Interrupt)**
    1. 예외, 입출력, 이벤트 등이 발생하여 현재 실행 중인 프로세스를 준비 상태로 바꾸고, 해당 작업을 먼저 처리하는 것.
4. **입출력 또는 이벤트 대기 (I/O or Event wait)**
    1. 실행 중인 프로세스가 입출력이나 이벤트를 처리해야 하는 경우, 입출력/이벤트가 모두 끝날 때까지 대기 상태로 만드는 것.
5. **입출력 또는 이벤트 완료 (I/O or Event Completion)**
    1. 입출력/이벤트가 끝난 프로세스를 준비 상태로 전환하여 스케줄러에 의해 선택될 수 있도록 만드는 것.

## CPU 스케줄링

- 비선점 스케줄링
    1. FCFS( First Come First Served )
        1. 큐에 도착한 순서대로 CPU 할당
        2. 실행 시간이 짧은 게 뒤로 가면 평균 대기 시간이 길어짐
    2. SJF( Shortest Job First )
        1. 수행 시간이 가장 짧다고 판단되는 작업 먼저 수행
        2. FCFS보다 평균 대기시간 감소 → 짧은 작업에 유리
    3. ⭐ HRN ( Highest Response-ratio Next)
        1. 우선순위를 계산하여 점유 불평들을 보완한 방법 (SJF의 단점 보완)
        2. 우선 순위 = (대기시간+실행시간) / (실행시간)

- 선점 스케줄링
    1. Priority Scheduling
        1. 정적/동적으로 우선순위를 부여하여 우선순위가 높은 순서대로 처리
        2. 우선 순위 낮음 → `starvation 발생`
        3. Aging 방법으로 starvation 문제 해결 가능
    2. Round Robin
        1. FCFS에 의해 프로세스들이 보내지면 각 프로세스는 동일한 시간의 `Time Quantum` 만큼 CPU를 할당 받음
        2. `Time Quantum` or `Time Slice` : 실행의 최소 단위 시간
        3. 할당 시간(`Time Quantum`)이 크면 FCFS와 같게 되고, 작으면 문맥 교환 (Context Switching) 잦아져서 오버헤드 증가
    3. Multilevel-Queue(다단계 큐)
        1. 프로세스들을 우선 순위 또는 작업 유형 등에 따라 여러 큐로 분류함
        2. 각 큐는 **독립적인 스케쥴링 알고리즘**을 가질 수 있음
        3. 큐를 분할하여 각 큐는 특정한 프로세스 집합을 처리한다.
            1. 예시
                1. **큐 1**: 인터랙티브 프로세스 (시간 민감 작업)
                2. **큐 2**: 배치 프로세스 (시간 민감하지 않은 작업)
         ![image (1)](https://github.com/user-attachments/assets/8c3b7634-8861-44f4-be92-c748bb6ea880)
         ![image (2)](https://github.com/user-attachments/assets/d3311904-ca94-4606-bd15-6535b29c3dcd)
        - 우선순위가 낮은 큐들이 실행 못하는 걸 방지하고자 각 큐마다 다른 `Time Quantum` (시간 할당량)을 설정 해주는 방식 사용
            - Time Quantum이란?
                - CPU가 각 큐에 할당하는 **시간의 크기**를 말함.
                - 큐마다 서로 다른 Time Quantum을 설정하여, 높은 우선순위의 작업이 계속 CPU를 독점하지 못하게 한다.
        - 우선순위가 높은 큐는 작은 `Time Quantum` 할당. 우선순위가 낮은 큐는 큰 `Time Quantum` 할당.
        - 따라서 각 큐가 교대로 CPU를 사용하며, 우선순위와 Time Quantum에 따라 공정하게 자원이 배분됨
    4. Multilevel-Feedback-Queue (다단계 피드백 큐)   
    ![image (3)](https://github.com/user-attachments/assets/8a0161f2-1355-4165-8436-88a7fbd44d9f)
        1. CPU 스케줄링 알고리즘 중 하나로, 프로세스가 **실행 중에 우선순위가 동적으로 변경**될 수 있는 방식이다.
        2. **Multilevel Queue**와 달리, **프로세스가 큐 간에 이동**하면서 작업 특성과 실행 시간에 맞게 CPU를 공정하게 배분할 수 있다.
        3. 기본 원리
            1. 프로세스는 처음 **우선순위가 높은 큐**에 배치된다.
            2. **Time Quantum**이 만료되거나 프로세스가 오래 실행되면 → 우선순위가 낮은 큐로 이동
            3. **입출력(I/O) 중심 프로세스**처럼 짧게 실행되거나 CPU 시간이 적게 필요한 프로세스는 높은 우선순위를 유지
            4. **작업 특성에 따라 큐를 이동**하면서 CPU를 효율적으로 사용
        4. 단점
            1. 설계 복잡성
            2. 프로세스의 우선순위를 조정하고 큐를 이동시키는 작업에서 추가 오버헤드가 발생
            3. 예측 어려움

### **CPU 스케줄링 척도**

1. Response Time
    - 작업이 처음 실행되기까지 걸린 시간
2. Turnaround Time
    - 실행 시간과 대기 시간을 모두 합한 시간으로 작업이 완료될 때 까지 걸린 시간
