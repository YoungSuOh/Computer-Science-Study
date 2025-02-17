# Java에서의 Thread

## Process & Thread

| 프로세스 | 스레드(Thread) |
| --- | --- |
| 운영체제로부터 자원을 할당 받은 `작업의 단위` | 프로세스가 할당받은 자원을 이용하는 `실행 흐름의 단위` |

- Program (Static Program)
    - 코드 덩어리임
    - 컴퓨터에서 실행할 수 있는 파일을 통칭한다.
- 프로세스 (Process)
    - 프로세스는 `프로그램`을 실행시켜 정적인 프로그램이 동적으로 변하여 프로그램이 돌아가고 있는 상태이다.
    - `ctrl + alt +del`을 통해 돌아가고 있는 프로세스를 쉽게 확인할 수 있다.
    - 모든 프로그램은 운영체제가 실행되기 위한 메모리 공간을 할당해 줘야 실행될 수 있다. 그래서 프로그램을 실행하는 순간은 파일은 컴퓨터 메모리에 올라가게 되고, 운영체제로부터 시스템 자원(CPU)을 할당 받아 프로그램 코드를 실행시켜 우리가 서비스를 이용할 수 있게 된다.
- 정리
    
    
    | **프로그램** | 프로세스 |
    | --- | --- |
    | 어떤 작업을 하기 위해 실행할 수 있는 파일 | 실행되어 작업중인 컴퓨터 프로그램 |
    | 파일이 저장 장치에 있지만 메모리에는 올라가 있지 않은 정적인 상태 | 메모리에 적재되고 CPU 자원을 할당받아 프로그램이 실행되고 있는 상태 |
    | 쉽게 말해 그냥 코드 덩어리 | 그 코드 덩어리를 실행한 것 |

- 프로세스의 한계
    - 과거에는 프로그램을 실행할 때, 프로세스 하나만을 사용해서 프로그램을 실행하기에는 한계가 있었음.
        - 하나 다운이 완료될때까지 하루종일 기다려야 했음
    - 오늘날 컴퓨터는 파일을 다운 받으며 다른 일을 하는 멀티 작업은 너무나 당연한 기능이 됨
    - Thead는 프로세스 특성의 한계를 해결하기 위해 탄생하였다.

- 쓰레드(Thread)
    - 하나의 프로세스 내에서 동시에 진행되는 작업 갈래, 흐름의 단위를 말한다.
        
        ![스크린샷(313).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/67522831-a1fd-4f80-963b-2bff0866dcb7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(313).png)
        
        - 예를 들면 위와 같이 크롬 브라우저가 실행되면, 프로세스 하나가 생성될 것이다. 그런데 우리는 브라우저에서 파일을 다운 받으며 온라인쇼핑을 하면서 게임을 하기도 한다.
        - 즉, 하나의 프로세스 안에서 여러가지 작업들 흐름이 동시에 진행되기 때문에 가능한 것인데, 이러한 일련의 작업 흐름들을 `스레드`라고 하며, 여러 개가 있다면 이를 `멀티 쓰레드` 라고 부른다.
    - 하나의 Process 내에 여러가지 쓰레드가 있는 모습이다.
        
        ![스크린샷(314).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/0306f30d-963e-429f-bade-ebde846fef76/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(314).png)
        
        - 하나의 프로그램은 하나 이상의 프로세스를 가지고 있고, 하나의 프로세스는 반드시 하나 이상의 스레드를 갖는다.
        - 즉, 프로세스를 생성하면 기본적으로 하나의 `main thread` 가 생성이 되고, 나머지 그 이상의 스레드는 프로그래밍을 통해 개발자가 직접 프로그래밍 해줘야 한다.

요즘 OS는 모두 멀티태스킹을 지원한다.

- 멀티태스킹이란?
    - 두 가지 이상의 작업을 동시에 하는 것을 말한다.

실제로 동시에 처리될 수 있는 프로세스의 개수는 CPU 코어의 개수와 동일한데,  이보다 많은 개수의 프로세스가 존재하기 때문에 모두 함께 동시에 처리할 수는 없다.

각 코어들은 아주 짧은 시간동안 여러 프로세스를 번갈아가며 처리하는 방식을 통해 동시에 동작하는 것처럼 보이게 할 뿐이다.

이와 마찬가지로, 멀티스레딩이란 하나의 프로세스안에 여러 개의 스레드가 동시에 작업을 수행하는 것을 말한다.

스레드는 하나의 작업 단위라 생각하면 편하다.

## 스레드 구현

자바에서 스레드 구현 방법은 2가지가 있다.

1. Runnable 인터페이스 구현
2. Thread 클래스 상속

둘 다 run()  메서드를 오버라이딩 하는 방식이다.

```java
public class MyThread implements Runnable {
    @Override
    public void run() {
        // 수행 코드
    }
}
```

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        // 수행 코드
    }
}
```

### Thread 인터페이스

```java
public class SnackMain extends Thread{
    private String str;

    public SnackMain(String str) {
        this.str = str;
    }

    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(str + "\t" + "i : " + i + "\t" + Thread.currentThread());
        }
    }

    public static void main(String[] args) {
        SnackMain aa = new SnackMain("새우깡");
        SnackMain bb = new SnackMain("포카칩");
        SnackMain cc = new SnackMain("수미칩");

        aa.setName("새우깡");
        bb.setName("포카칩");
        cc.setName("수미칩");

        aa.start();		//스레드 시작 -> 운영체제가 알아서 스레드 실행)

        // 값을 항상 전의 것을 기억하고 있기 때문에 CPU, 메모리 많이 잡아먹음.
        try {
            aa.join();	// 스레드가 끝날때까지 기다리는 메소드
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        bb.start();
        cc.start();

//		aa.run();		//(운영체제가 알아서 run() 호출, 콜백메소드(Call Back)
//		bb.run();
//		cc.run();
    }

}

result :
새우깡	i : 1	Thread[새우깡,5,main]
새우깡	i : 2	Thread[새우깡,5,main]
새우깡	i : 3	Thread[새우깡,5,main]
새우깡	i : 4	Thread[새우깡,5,main]
새우깡	i : 5	Thread[새우깡,5,main]
포카칩	i : 1	Thread[포카칩,5,main]
포카칩	i : 2	Thread[포카칩,5,main]
포카칩	i : 3	Thread[포카칩,5,main]
포카칩	i : 4	Thread[포카칩,5,main]
포카칩	i : 5	Thread[포카칩,5,main]
수미칩	i : 1	Thread[수미칩,5,main]
수미칩	i : 2	Thread[수미칩,5,main]
수미칩	i : 3	Thread[수미칩,5,main]
수미칩	i : 4	Thread[수미칩,5,main]
수미칩	i : 5	Thread[수미칩,5,main]
```

- `SnackMain` 클래스는 `Thread` 클래스를 상속 받는다. 이로 인해 `SnackMain` 객체는 독립적인 스레드로 실행될 수 있다.
- 스레드가 시작되었을 때 `run` 메소드가 실행이 된다.
- `Thread` 클래스의 `start` 메서드가 호출이 되면 자동으로 `run` 메서드가 호출됩니다.
- 처음에 스레드 `aa` 가 시작하고, 그  이후 `aa.join()` 메서드를 호출하여 `aa` 스레드가 끝날 때까지 `main` 스레드가 기다린다.
- 이후 `bb`와 `cc` 스레드를 시작한다.

## Runnable 인터페이스

```java
class JoinTest implements Runnable{

    @Override
    public void run() {		//Call-Back 메소드(콜백메소드, 운영체제에 의해 불려지는 메소드)
        for (int i = 1; i <= 5; i++) {
            System.out.println("i : " + i + "\t" + Thread.currentThread());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

- Runnable 인터페이스의 추상 메서드를 override 했다.
- for문을 돌면서 현재 스레드와 반복 인덱스를 출력하는데, 돌때마다 1초동안 대기한다.

```java
public class JoinMain {

    public static void main(String[] args) {
        JoinTest joinTest = new JoinTest();
        Thread t = new Thread(joinTest);  // 스레드 생성

        System.out.println("스레드 시작");
        t.start();  // 스레드 시작
        try {
            t.join();  // 스레드가 끝날 때까지 기다리는 메소드
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("스레드 종료");
    }

}

```

## Synchroinze

- 동기화 처리 : 여러 객체가 동시에 하나의 메서드에 접근하려고 할 때 심각한 부하가 발생된다. → 이를 방지하기 위해 lock을 건다. ⇒ 이를 동기화 처리라고 함(synchronized)
- 동기화 x 예시
    
    ```java
    class Synchornized extends Thread{
        private static int count;
        @Override
        public void run(){       
            for(int i=1;i<=5;i++){
                count++;
                System.out.println(Thread.currentThread()+" : "+count);
            }
    
        }
    
    }
    
    public class SynchronizedMain {
        public static void main(String[] args) {
            Synchornized aa = new Synchornized(); // 쓰레드 생성
            Synchornized bb = new Synchornized(); // 쓰레드 생성
            Synchornized cc = new Synchornized(); // 쓰레드 생성
    
            aa.setName("aa");
            bb.setName("bb");
            cc.setName("cc");
    
            // 스레드 시작 -> 스레드 실행( run() 메서드 찾아감 )
            aa.start();
            bb.start();
            cc.start();
    
        }
    }
    result: -> 결과 뒤죽박죽
    Thread[cc,5,main] : 3
    Thread[aa,5,main] : 1
    Thread[aa,5,main] : 5
    Thread[aa,5,main] : 6
    Thread[aa,5,main] : 7
    Thread[aa,5,main] : 8
    Thread[cc,5,main] : 4
    Thread[cc,5,main] : 9
    Thread[bb,5,main] : 2
    Thread[cc,5,main] : 10
    Thread[cc,5,main] : 12
    Thread[bb,5,main] : 11
    Thread[bb,5,main] : 13
    Thread[bb,5,main] : 14
    Thread[bb,5,main] : 15
    
    ```
    
    - `스레드 프로그래밍` 에서는 각 스레드의 실행 순서는 OS의 스케쥴러에 의해 관리된다.
    - 즉, 스레드가 실행되는 순서와 각 스레드가 실행되는 시점은 예측할 수 없음.
    - 세 개의 스레드 `aa`, `bb`, `cc`가 거의 동시에 시작되기 때문에 실행 순서가 뒤섞인다.
    - `count` 변수를 증가시키는 부분에서 `동기화가 이루어지지 않기 때문에`여러 스레드가 동시에 `count` 변수를 수정하려고 할 때, 의도하지 않은 결과가 발생할 수 있다.
- 클래스 수준의 동기화
    
    ```java
    class Synchornized extends Thread{
        private static int count;
    
        @Override
        public void run(){       
            synchronized (Synchornized.class) {
                for(int i=1;i<=5;i++){
                    count++;
                    System.out.println(Thread.currentThread()+" : "+count);
                }
            } 
        }
    }
    result:
    Thread[aa,5,main] : 1
    Thread[aa,5,main] : 2
    Thread[aa,5,main] : 3
    Thread[aa,5,main] : 4
    Thread[aa,5,main] : 5
    Thread[cc,5,main] : 6
    Thread[cc,5,main] : 7
    Thread[cc,5,main] : 8
    Thread[cc,5,main] : 9
    Thread[cc,5,main] : 10
    Thread[bb,5,main] : 11
    Thread[bb,5,main] : 12
    Thread[bb,5,main] : 13
    Thread[bb,5,main] : 14
    Thread[bb,5,main] : 15
    ```
    
    - `Synchornized.class` 객체에 대해 동기화를 하고 있다.
    - `synchronized (Synchornized.class)` 블록은 해당 클래스의 모든 인스턴스가 이 블록에 들어갈 때 동일한 모니터 객체를 사용하게 한다.
        - 여기서 `모니터 객체`란?
            - 자바에서 모든 객체는 monitor를 가지고 있다.
            - monitor는 여러 스레드가 객체로 동시에 객체로 접근하는 것을 막는다
            - 여기서 모든 객체가 중요하다. heap 영역에 있는 객체는 모든 스레드에서 공유 가능하기 때문이다.
            - 스레드가 monitor를 가지면, monitor를 가지는 객체에 lock을 걸 수 있다
            - 그렇게 되면 다른 thread들이 해당 객체에 접근할 수가 없게된다.
    - 즉, 어떤 스레드가 이 블록을 실행하는 동안 다른 스레드는 이 블록에 들어갈 수 없다.

- 인스턴스 수준의 동기화
    
    ```java
    class Synchornized extends Thread{
        private static int count;
    
        @Override
        public void run(){       
            synchronized (this) { 
                for(int i=1;i<=5;i++){
                    count++;
                    System.out.println(Thread.currentThread()+" : "+count);
                }
            }
        }
    }
    result:
    Thread[cc,5,main] : 3
    Thread[bb,5,main] : 2
    Thread[bb,5,main] : 5
    Thread[bb,5,main] : 6
    Thread[bb,5,main] : 7
    Thread[bb,5,main] : 8
    Thread[aa,5,main] : 1
    Thread[cc,5,main] : 4
    Thread[cc,5,main] : 10
    Thread[cc,5,main] : 11
    Thread[cc,5,main] : 12
    Thread[aa,5,main] : 9
    Thread[aa,5,main] : 13
    Thread[aa,5,main] : 14
    Thread[aa,5,main] : 15
    ```
    
    - `this` 객체에 대해 동기화를 하고 있다.
    - `synchronized (this)` 블록은 해당 인스턴스에 대해서만 동기화를 적용합니다. 따라서 서로 다른 인스턴스는 이 블록을 동시에 실행할 수 있다.
    - `count` 변수는 `static`으로 선언되어 있으므로, 실제로는 모든 인스턴스가 동일한 `count` 변수를 공유한다.
    - 이는 `count` 변수의 동기화가 제대로 이루어지지 않게 됩니다. 따라서 인스턴스 수준의 동기화는 이 경우 올바른 동기화 방법이 아니다.

- 우선 순위 부여
    
    ```java
    public class SynchronizedMain {
        public static void main(String[] args) {
            Synchornized aa = new Synchornized(); // 쓰레드 생성
            Synchornized bb = new Synchornized(); // 쓰레드 생성
            Synchornized cc = new Synchornized(); // 쓰레드 생성
    
            aa.setName("aa");
            bb.setName("bb");
            cc.setName("cc");
    
            // 우선 순위 부여 1~ 10까지
    
            aa.setPriority(1);
            aa.setPriority(Thread.MIN_PRIORITY); // = aa.setPriority(1);
            bb.setPriority(10); //  bb.setPriority(Thread.MAX_PRIORITY);
            cc.setPriority(5);
    
            // 스레드 시작 -> 스레드 실행( run() 메서드 찾아감 )
            aa.start();
            bb.start();
            cc.start();
    
        }
    }
    
    ```
    

- 싱글톤
    - DB에는 싱글톤을 걸어줘야 한다.
    - 싱글톤 x 예제
        
        ```java
        class Singleton{
            private int num = 3;
            private static Singleton singletonInstance;
            public void calc(){
                num++;
            }
            public void disp(){
                System.out.println("num = " + num);
            }    
        }
        
        public class SingletonMain {
            public static void main(String[] args) {
                Singleton aa =new Singleton();  -> 싱글톤 지원 안함
                System.out.println(aa);
                aa.calc();
                aa.disp();
                System.out.println();
        
                Singleton bb =new Singleton();
                System.out.println(bb);
                bb.calc();
                bb.disp();
                System.out.println();
        
                Singleton cc =new Singleton();
                System.out.println(cc);
                cc.calc();
                cc.disp();
                System.out.println();
        
                result :
                thread.Singleton@3b07d329
        
                num = 4
        
                thread.Singleton@6d03e736
                num = 4
        
                thread.Singleton@568db2f2
                num = 4
                
        
             
            }
        }
        ```
        
    - 싱글톤 지원 o
        
        ```java
        class Singleton{
            private int num = 3;
            private static Singleton singletonInstance;
            public void calc(){
                num++;
            }
            public void disp(){
                System.out.println("num = " + num);
            }
            public static Singleton getInsatance(){
                if(singletonInstance==null){
                    synchronized (Singleton.class){
                        singletonInstance = new Singleton();
                    }
                }
                return singletonInstance;
            }
        }
        
        public class SingletonMain {
            public static void main(String[] args) {    
        
                // Singleton 적용 후
                Singleton aa = Singleton.getInsatance();
                System.out.println(aa);
                aa.calc();
                aa.disp();
                System.out.println();
        
                Singleton bb =Singleton.getInsatance();
                System.out.println(bb);
                bb.calc();
                bb.disp();
                System.out.println();
        
                Singleton cc =Singleton.getInsatance();
                System.out.println(cc);
                cc.calc();
                cc.disp();
                System.out.println();
            }
        }
        result:
        thread.Singleton@448139f0
        num = 4
        
        thread.Singleton@448139f0
        num = 5
        
        thread.Singleton@448139f0
        num = 6
        
        ```