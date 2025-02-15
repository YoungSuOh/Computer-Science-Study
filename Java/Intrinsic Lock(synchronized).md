Java에서 **고유 락(Intrinsic Lock)** 또는 **모니터 락(Monitor Lock)**은 **`synchronized` 키워드**를 사용할 때 자동으로 적용되는 락을 의미한다. 

Java의 모든 객체(Object)에는 기본적으로 고유 락이 내장되어 있으며, `synchronized`를 사용하면 해당 객체의 락을 획득하고 임계 영역(critical section)에 대한 접근을 제어할 수 있다.

- Critical Section
    - Critical Section(임계 영역)은 **멀티스레드(또는 멀티프로세스) 환경에서 공유 자원에 접근하는 코드 영역**
    - 즉, 여러 개의 스레드(또는 프로세스)가 동시에 실행될 때, **공유 자원을 두 개 이상의 스레드가 접근하면 충돌이 발생할 가능성이 있는 영역**

## 특징

1. 객체 단위로 관리됨
    - 모든 Java 객체에는 고유 락이 있으며, `synchronized`를 사용하면 해당 객체의 락을 획득함.
    - 한 스레드가 락을 획득하면 다른 스레드는 해당 락이 해제될 때까지 대기해야 함.
2. 자동 해제
    - `synchronized` 블록이나 메서드가 종료되면 자동으로 락이 해제됨
    - 예외가 발생해도 은 해제됨 (즉, `finally` 블록에서 별도로 해제할 필요 없음)
3. **재진입 가능(Reentrant)**
- 같은 스레드가 동일한 객체의 `synchronized` 블록을 여러 번 호출하면, 락을 다시 획득할 수 있음.
- 즉, 한 스레드가 특정 객체의 락을 가지고 있는 동안, 다시 해당 객체의 락이 필요한 경우에도 문제없이 접근 가능.

## 사용법

1. synchronized 메서드
    
    ```java
    class Counter {
        private int count = 0;
    
        public synchronized void increment() {  // 객체의 고유 락을 획득
            count++;
        }
    
        public synchronized int getCount() {  // 고유 락이 해제될 때까지 다른 스레드는 접근 불가
            return count;
        }
    }
    ```
    
    - `synchronized` 메서드는 **해당 객체(`this`)의 고유 락을 획득**해야 실행 가능.
    - 한 스레드가 `increment()`를 실행 중이면, 다른 스레드는 `getCount()`를 실행할 수 없음.
2. synchronized 블록
    
    ```java
    class Counter {
        private int count = 0;
    
        public void increment() {
            synchronized (this) {  // 명시적으로 객체(this)의 고유 락을 획득
                count++;
            }
        }
    
        public int getCount() {
            synchronized (this) {
                return count;
            }
        }
    }
    
    ```
    
    - `synchronized` 블록을 사용하면 특정 코드 영역만 보호할 수 있어 더 유연하게 활용 가능.
    - `this` 대신 특정 객체를 지정하여 고유 락을 관리할 수도 있음.
3. static 메서드에서의 고유 락
    
    ```java
    class SharedResource {
        private static int count = 0;
    
        public static synchronized void increment() {
            count++;
        }
    
        public static synchronized int getCount() {
            return count;
        }
    }
    ```
    
    - `static synchronized` 메서드는 **클래스(Class) 수준의 락**을 사용함.
    - 따라서 `SharedResource` 클래스의 모든 인스턴스가 공유하는 락을 획득함.

## 단점

1. 성능 저하
    - `synchronized`를 사용하면 스레드가 순차적으로 실행되므로 성능이 저하될 수 있음.
    - 불필요한 동기화는 피해야 함.
2. 교착 상태(Deadlock) 발생 가능
    - 여러 개의 락을 사용할 경우 교착 상태가 발생할 수 있음.
    
    ```java
    class Resource {
        public void method1() {
            synchronized (this) {
                System.out.println("Method 1 실행 중...");
                method2();  // 같은 객체(this)의 락을 다시 획득해야 함 (재진입 가능)
            }
        }
    
        public void method2() {
            synchronized (this) {
                System.out.println("Method 2 실행 중...");
            }
        }
    }
    ```
    
    - 위의 경우는 같은 객체에 대한 락을 재진입하므로 문제없지만, **두 개 이상의 객체를 락으로 사용하면 교착 상태 위험이 있음**

- DeadLock 발생 예시
    
    ```java
    class DeadlockExample {
        private final Object lock1 = new Object();
        private final Object lock2 = new Object();
    
        public void method1() {
            synchronized (lock1) {  // lock1을 획득
                System.out.println("Method1: lock1 획득");
    
                try { Thread.sleep(100); } catch (InterruptedException e) {}
    
                synchronized (lock2) {  // lock2를 획득하려고 시도 (Deadlock 발생 가능)
                    System.out.println("Method1: lock2 획득");
                }
            }
        }
    
        public void method2() {
            synchronized (lock2) {  // lock2를 먼저 획득
                System.out.println("Method2: lock2 획득");
    
                try { Thread.sleep(100); } catch (InterruptedException e) {}
    
                synchronized (lock1) {  // lock1을 획득하려고 시도 (Deadlock 발생 가능)
                    System.out.println("Method2: lock1 획득");
                }
            }
        }
    }
    
    public class DeadlockTest {
        public static void main(String[] args) {
            DeadlockExample example = new DeadlockExample();
    
            Thread t1 = new Thread(example::method1);
            Thread t2 = new Thread(example::method2);
    
            t1.start();
            t2.start();
        }
    }
    ```
    
    - `method1()`에서 `lock1`을 획득한 후 `lock2`를 기다림.
    - `method2()`에서 `lock2`를 획득한 후 `lock1`을 기다림.
    - 서로의 락을 기다리면서 무한 대기(Deadlock) 상태 발생.

## 대체 방법

고유 락을 사용하지 않고 동기화를 개선할 수 있는 방법

1. ReentrantLock (명시적 락)
    - `synchronized` 대신 `java.util.concurrent.locks.ReentrantLock`을 사용하면 보다 유연하게 락을 제어할 수 있음.
    - 락 획득과 해제를 명시적으로 제어 가능.
    - **tryLock(), lockInterruptibly()** 등의 기능을 제공하여 교착 상태를 방지할 수 있음.
    
    ```java
    import java.util.concurrent.locks.ReentrantLock;
    
    class SharedResource {
        private final ReentrantLock lock = new ReentrantLock();
        private int count = 0;
    
        public void increment() {
            lock.lock();  // 락 획득
            try {
                count++;
            } finally {
                lock.unlock();  // 반드시 락 해제
            }
        }
    
        public int getCount() {
            return count;
        }
    }
    ```
    
    - `finally` 블록에서 **반드시 락을 해제**하므로 예외 발생 시에도 안전.
2. tryLock()
    - `ReentrantLock`에서 `tryLock()`을 사용하면 **락이 사용 중일 때 대기하지 않고 즉시 반환**됨.
    
    ```java
    import java.util.concurrent.locks.ReentrantLock;
    
    class TryLockExample {
        private final ReentrantLock lock = new ReentrantLock();
    
        public void safeMethod() {
            if (lock.tryLock()) {  // 락을 즉시 시도하고 실패하면 실행하지 않음
                try {
                    System.out.println("락 획득 성공");
                } finally {
                    lock.unlock();
                }
            } else {
                System.out.println("락을 획득하지 못함");
            }
        }
    }
    
    ```
    
    - **교착 상태(Deadlock)를 방지**할 수 있음.
    - **락이 걸려 있으면 즉시 반환**하여 다른 작업을 수행할 수 있음
3. **lockInterruptibly()**
    - `lockInterruptibly()`는 **스레드가 대기하는 동안 인터럽트를 받을 수 있도록** 함.
    - 교착 상태가 발생할 가능성이 있는 경우, 다른 스레드에서 인터럽트를 걸어 해제할 수 있음.
    
    ```java
    import java.util.concurrent.locks.ReentrantLock;
    
    class InterruptibleLockExample {
        private final ReentrantLock lock = new ReentrantLock();
    
        public void safeMethod() {
            try {
                lock.lockInterruptibly();  // 인터럽트 가능하게 락 획득 시도
                try {
                    System.out.println("작업 수행 중...");
                } finally {
                    lock.unlock();
                }
            } catch (InterruptedException e) {
                System.out.println("인터럽트 감지, 작업 취소");
            }
        }
    }
    
    ```
    
4. volatile
    - `volatile` 키워드는 **변수 값이 즉시 다른 스레드와 공유됨**을 보장.
    - 하지만 **단순한 읽기/쓰기 연산만 원자적(atomic)으로 보장되며, 복잡한 동기화에는 적합하지 않음**.
    
    ```java
    class VolatileExample {
        private volatile boolean flag = false;
    
        public void updateFlag() {
            flag = true;  // 모든 스레드가 즉시 변경 사항을 인식
        }
    
        public boolean getFlag() {
            return flag;
        }
    }
    
    ```
    
5. Atomic 변수
    - `AtomicInteger`, `AtomicLong` 등은 **락 없이도 동기화된 연산을 제공**하는 클래스
    
    ```java
    import java.util.concurrent.atomic.AtomicInteger;
    
    class AtomicExample {
        private AtomicInteger count = new AtomicInteger(0);
    
        public void increment() {
            count.incrementAndGet();  // thread-safe 증가 연산
        }
    
        public int getCount() {
            return count.get();
        }
    }
    
    ```