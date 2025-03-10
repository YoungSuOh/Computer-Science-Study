# Cache

캐시(Cache)는 **자주 사용하는 데이터를 빠르게 접근할 수 있도록 임시로 저장하는 메모리 공간이다.**

CPU와 RAM(주기억장치) 간 속도 차이를 줄이기 위해 사용되며, 컴퓨터뿐만 아니라 웹 브라우저, 데이터베이스 등 다양한 시스템에서도 활용된다.

## 왜 캐시가 필요한가?

⚡ CPU vs RAM 속도 차이

- **CPU 연산 속도**는 **나노초(ns, 10⁻⁹초) 단위**로 엄청 빠르지만,
- **RAM 접근 속도**는 상대적으로 느려서 **마이크로초(µs, 10⁻⁶초) 단위**가 걸려.
- 즉, **CPU가 데이터를 요청할 때마다 RAM에서 가져오면 성능이 크게 저하**됨.
- 그래서 **CPU 내부에 속도가 훨씬 빠른 캐시(Cache)를 둬서 자주 쓰는 데이터를 미리 저장**하는 거야.

## 캐시의 동작 원리

1. CPU가 데이터 요청
    - CPU는 먼저 캐시 메모리에서 필요한 데이터가 있는지 확인
2. 캐시에 데이터가 있으면 Hit
    - 바로 캐시에서 가져오므로 속도가 매우 빠
3. 캐시에 없으면 Miss
    - RAM에서 데이터를 가져와야 해서 시간이 걸림.
    - 가져온 데이터를 **캐시에 저장**해두고, 이후에 같은 요청이 오면 빠르게 처리할 수 있도록 함.

🔹 이때 캐시 히트(Cache Hit)가 많을수록 성능이 좋아지고,

🔹 캐시 미스(Cache Miss)가 많으면 속도 저하가 발생함.

## 캐시 메모리의 계층 구조

캐시는 보통 **L1, L2, L3** 계층 구조로 나뉘며, **L1이 가장 빠르고 작으며, L3가 가장 크고 느림**.

**L1, L2, L3 캐시는 CPU가 직접 관리하며, 속도 차이를 줄이기 위한 계층 구조를 형성**함.

## 캐시 교체 알고리즘 (Cache Replacement Policy)

1. LRU(Least Recently Used, 가장 오랫동안 사용되지 않은 데이터 제거
2. LFU(Least Frequently Used, 가장 적게 사용된 데이터 제거)
3. FIFO(First In First Out, 먼저 들어온 데이터 제거)

## 캐시의 활용 분야

### **1. CPU 캐시**

- **CPU와 RAM 간 속도 차이를 줄이기 위해 사용**됨.
- **L1 → L2 → L3 순으로 캐시를 검색**해서 데이터를 빠르게 가져옴.

### **2. 웹 브라우저 캐시**

- 웹사이트의 이미지, CSS, JavaScript 등을 **로컬 저장소에 저장**해서 로딩 속도를 높임.
- **Ctrl + F5**를 누르면 강제로 캐시를 삭제하고 새로고침 가능.

### **3. 데이터베이스 캐시**

- **Redis, Memcached 같은 인메모리 캐시를 사용**해서 DB 쿼리 속도를 향상시킴.
- 예를 들어, **자주 조회되는 데이터를 캐시에 저장해서 DB 부하를 줄임**.

### **4. CDN (Content Delivery Network) 캐시**

- **전 세계 여러 지역에 캐시 서버를 배치**해서 사용자에게 가까운 서버에서 콘텐츠를 제공.
- 이를 통해 **웹사이트 로딩 속도 개선 & 서버 부하 감소** 효과가 있음.

## 캐시 메모리의 쓰기(Write) 정책

CPU가 데이터를 수정할 때, **캐시에만 반영할지(RAM은 나중에 업데이트)** 또는 **RAM과 동시에 반영할지**를 결정하는 정책이 있다.

### **1. Write-Through (쓰기-통과)**

### 📌 개념

- **캐시와 RAM에 동시에 데이터를 기록**하는 방식.
- 데이터의 일관성이 보장되지만, **쓰기 성능이 느려질 수 있음**.

### 📌 장점

✔ **데이터 일관성 유지** (항상 RAM과 동일한 값을 가짐)

✔ 캐시가 날아가도 **RAM에 최신 데이터가 있어서 복구 가능**

### 📌 단점

❌ **쓰기 속도가 느림** (매번 RAM까지 업데이트하므로)

❌ **불필요한 메모리 접근 증가** (자주 변경되는 데이터도 항상 RAM까지 기록)

### **2. Write-Back (쓰기-되돌림)**

### 📌 개념

- 데이터를 **먼저 캐시에만 저장**하고,
- 나중에 해당 데이터가 **캐시에서 제거될 때만 RAM에 기록**하는 방식.

### 📌 장점

✔ **쓰기 속도가 빠름** (RAM까지 매번 접근할 필요 없음)

✔ **불필요한 RAM 접근 최소화**

### 📌 단점

❌ **전력 손실이나 시스템 오류 시 데이터 손실 가능성 있음** (RAM에 즉시 기록되지 않음)

❌ **데이터 일관성 관리가 어려움** (캐시 데이터와 RAM 데이터가 다를 수 있음)

## 캐시 블록 할당 정책 (Cache Block Allocation Policy)

CPU가  **캐시에 없는 데이터를 요청했을 때,** RAM에서 데이터를 가져오는 방법을 결정하는 정책이다.

### **1. Write Allocate (쓰기-할당)**

📌 **특징**

- 데이터를 **캐시에 먼저 로드한 후, 수정 작업 수행**
- 즉, **캐시 미스(Cache Miss)가 발생하면 먼저 캐시에 데이터를 가져오고, 이후에 쓰기 작업 진행**
- 주로 **Write-Back 방식과 함께 사용**됨.

📌 **장점**

✔ 캐시 적중률(Cache Hit Ratio) 증가 (데이터를 캐시에 유지)

📌 **단점**

❌ 불필요한 캐시 공간 사용 가능

### **2. No-Write Allocate (비-쓰기 할당)**

📌 **특징**

- **캐시에 없는 데이터(미스)일 때 쓰기 연산 발생** → **RAM에 바로 씀** (캐시에 올리지 않음)
    - 즉, **캐시 미스가 발생해도 RAM에서 직접 데이터를 수정하고, 캐시는 그대로 둠**.
- 읽기 연산은 여전히 캐시에 저장

📌 **장점**

✔ 캐시 공간 절약 (자주 사용되지 않는 데이터가 캐시에 불필요하게 차지하는 걸 방지)

📌 **단점**

❌ 캐시 적중률(Cache Hit Ratio)이 낮아질 수 있음 (RAM에만 데이터가 존재하기 때문에)

### 조합 정리

| 조합 | 특징 | 일반적인 사 |
| --- | --- | --- |
| Write-Back + No-Write Allocate | RAM 접근을 최소화하고, 쓰기 성능을 향상 | 가장 일반적인 조합 |
| Write-Back + Write Allocate | 쓰기 데이터를 캐시에 유지하여 적중률 향상 | 일부 CPU에서 사용 |
| Write-Through + Write Allocate | 항상 RAM에 저장하여 데이터 일관성 유지 | 일관성이 중요한 시스템 (ex. 데이터베이스) |
| Write-Through + No-Write Allocate | 캐시 오염 방지, 하지만 RAM 접근이 많음 | 특수한 경우 |