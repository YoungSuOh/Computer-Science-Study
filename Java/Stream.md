# Stream

자바 8 버전 이상부터 람다를 사용한 함수형 프로그래밍이 가능해졌다.

기존에 존재하던 Collection과 Stream은 무슨 차이가 있을까? 바로 **데이터 계산 시점**이다.

**Collection**

- **Collection**은 기본적으로 **데이터를 저장하고 관리하는 용도이다.**
- 모든 값을 메모리에 저장하는 자료구조다. 따라서 Collection에 추가하기 전에 미리 계산이 완료되어 있어야 한다.
- 외부 반복을 통해 사용자가 직접 반복 작업을 거쳐 요소를 가져올 수 있다.  (for-each)

**Stream**

- **Stream**은 데이터를 **처리하는 데 초점**을 맞춘 API이다.
- 요청할 때만 요소를 계산한다. 내부 반복을 사용하므로, 추출 요소만 선언해주면 알아서 반복 처리를 진행한다.
- 스트림에 요소를 따로 추가 혹은 제거하는 작업은 불가능하다.

```java
List<String> list = Arrays.asList("A", "B", "C");

// Collection 방식: 직접 요소를 순회하면서 처리
for (String s : list) {
    System.out.println(s.toLowerCase());
}

// Stream 방식: 선언적으로 처리
list.stream()
    .map(String::toLowerCase)
    .forEach(System.out::println);
```

## 외부 반복 & 내부 반복

Collection은 외부 반복, Stream은 내부 반복이라고 했다. 두 차이를 알아봅세~

**성능면에서는 내부 반복이 비교적 좋다**. 내부 반복은 작업을 병렬 처리하면서 최적화된 순서로  처리해준다.

하지만 외부 반복은 명시적으로 컬렉션 항목을 하나씩 가져와서 처리해야 하기 때문에 최적화에 불리하다.

즉, Collection에서 병렬성을 이용하려면 직접 `synchronized` 를 통해 관리해야 한다.

![image (33)](https://github.com/user-attachments/assets/7a386dc0-5e34-4b94-b87a-7339208c08bd)


### 🎯 **핵심 차이**

|  | 외부 반복 (for-each) | 내부 반복 (stream) |
| --- | --- | --- |
| 반복 제어 | 직접 제어 (개발자가 `for` 문으로 처리) | 자동 제어 (Stream이 반복 처리) |
| 병렬 처리 | 기본적으로 불가능 (수동으로 `synchronized` 해야 함) | 가능 (내부적으로 최적화됨) |
| 성능 | 단일 스레드로 동작 | 병렬 스트림을 사용하면 성능 향상 가능 |

## 💡 **CPU 연산 관점에서 내부 반복 vs 외부 반복 차이**

내부 반복과 외부 반복은 **반복을 실행하는 주체**가 다르기 때문에, **CPU 연산 방식도 차이가 있다.**

1️⃣ 외부 반복 (for-each)

💻 **CPU가 직접 하나씩 데이터를 꺼내서 연산을 수행**

👉 **단일 스레드에서 순차적으로 처리됨**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 외부 반복 (직접 for문 사용)
int sum = 0;
for (int num : numbers) {
    sum += num;
}
System.out.println(sum);
```

✅ **CPU 연산 방식**

- **데이터를 하나씩 가져온다 (메모리 접근)**
- **연산을 수행한다 (CPU 계산)**
- **다음 데이터를 가져와서 반복한다**
- **순차적으로 반복하면서 CPU가 하나씩 처리 (Single Thread)**

2️⃣ 내부 반복 (Stream)

💻 **CPU가 반복을 직접 수행하지 않고, 스트림 연산을 최적화하여 처리**

👉 **연산을 "필요할 때만" 실행 → 병렬 처리가 가능함**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 내부 반복 (Stream 사용)
int sum = numbers.stream()
                 .mapToInt(Integer::intValue)
                 .sum();

System.out.println(sum);
```

### ✅ **CPU 연산 방식**

- **데이터를 한꺼번에 처리할 준비 (스트림 생성)**
- **연산이 필요한 순간에만 데이터를 가져옴 (Lazy Evaluation)**
- **필요하면 병렬 처리하여 여러 개의 CPU 코어를 활용 (Parallel Stream 가능)**
- **최적화된 순서로 연산을 수행 (Pipeline 처리)**

## Stream 연산

스트림은 연산 과정이 ‘중간’과 ‘최종’으로 나누어진다.

`filter, map, limit` 등 파이프라이닝이 가능한 연산을 중간 연산, `count, collect` 등 스트림을 닫는 연산을 최종 연산이라고 한다.

둘로 나누는 이유는, 중간 연산들은 스트림을 반환해야 하는데, 모두 한꺼번에 병합하여 연산을 처리한 다음 최종 연산에서 한꺼번에 처리하게 된다.

ex) Item 중에 가격이 1000 이상인 이름을 5개 선택한다.

```java
List<String> items = item.stream()
    			.filter(d->d.getPrices()>=1000)
                          .map(d->d.getName())
                          .limit(5)
                          .collect(tpList());
```

- filter와 map은 다른 연산이지만, 한 과정으로 병합된다.
- 만약 Collection 이었다면, 우선 가격이 1000 이상인 아이템을 찾은 다음, 이름만 따로 저장한 뒤 5개를 선택해야 한다. 연산 최적화는 물론, 가독성 면에서도 Stream이 더 좋다.
