## JVM

자바 코드를 직접 실행하는 게 아니라, **바이트코드(.class 파일)** 를 해석하고 실행하는 역할을 해.
⇒ 즉, OS에 독립적으로 동작할 수 있도록 하는 핵심기 

## JVM의 동작 과정

1. **Java 소스 코드(.java) 작성**
    - 개발자가 Java 언어로 작성한 코드.
2. **컴파일(.class 파일 생성)**
    - `javac`(Java Compiler)가 `.java` 파일을 **바이트코드(.class)** 로 변환.
3. **JVM이 바이트코드 실행**
    - OS와 상관없이 바이트코드를 실행할 수 있도록 해석(인터프리팅 or JIT 컴파일).
4. **OS에 맞게 실행**
    - JVM이 OS와 상호작용하며 프로그램을 실행.

## JVM의 구조도

![image.png](attachment:f41d8bbb-dc55-43e7-add5-1c2271c34fe8:image.png)

1. JVM이 프로그램 실행을 위해 OS로부터 메모리를 할당받음
    - Java 프로그램이 실행되면 **JVM 프로세스가 생성**되고, 운영체제(OS)로부터 **Heap, Stack, Method Area** 등 필요한 메모리를 할당받음
2. Java 소스 코드 컴파일 (Javac) → 바이트코드 생성
    - **Java 컴파일러(Javac)** 가 `.java` 파일을 읽고 `.class` 바이트코드 파일로 변환함
3. 클래스 로더 (Class Loader) → JVM 메모리로 로드
    - 클래스 파일을 **JVM의 메모리(Runtime Data Area)**에 올려서 실행할 준비를 한다.
        - 이때, 로딩 → 링킹 → 초기화 과정을 진행한다.
4. 실행 엔진(Execution Engine) → 바이트코드 해석 및 실행
    - 인터프리터 방식 (Interpreter) : 바이트코드를 한 줄씩 해석하며 실행 (속도가 느림)
    - JIT(Just-In-Time) 컴파일러 방식 : 자주 실행되는 코드를 **기계어로 변환하여 캐싱**하고, 다음 실행부터 빠르게 실행
5. 메모리 관리 및 가비지 컬렉션
6. 최종적으로 프로그램 실행 결과 출력

## **가비지 컬렉션(Garbage Collection)**

자바 이전에는 프로그래머가 모든 프로그램 메모리를 관리했음 → new delete를 통해 해제

하지만, 자바에서는 `JVM`이 프로그램 메모리를 관리함

JVM은 가비지 컬렉션이라는 프로세스를 통해 메모리를 관리함. 가비지 컬렉션은 자바 프로그램에서 사용되지 않는 메모리를 지속적으로 찾아내서 제거하는 역할을 함.

**실행순서**

- 참조되지 않은 객체들을 탐색 후 삭제 → 삭제된 객체의 메모리 반환 → 힙 메모리 재사용

## 추가 질문

- Why 가상 머신??
    - JVM은 **하드웨어(CPU, 메모리 등) 위에서 직접 실행되는 물리적인 머신이 아니라, 소프트웨어적으로 구현된 머신**
    - 운영체제(OS)와 독립적
    - 바이트코드를 해석하여 실행
    - 메모리 관리 및 Garbage Collection(GC) 수행

- 바이트코드(Bytecode)란?
    - **Java 소스 코드(.java)** 를 **Javac(Java Compiler)** 가 변환한 **중간 코드(.class)** 이다.
    - 예시
        - Java 코드 작성
            
            ```java
            public class HelloWorld {
                public static void main(String[] args) {
                    System.out.println("Hello, World!");
                }
            }
            ```
            
        - 바이트코드 확인 (`javap -c HelloWorld.class`)
            
            ```java
            public static void main(java.lang.String[]);
              Code:
                 0: getstatic     #2 // Field java/lang/System.out:Ljava/io/PrintStream;
                 3: ldc           #3 // String Hello, World!
                 5: invokevirtual #4 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
                 8: return
            
            ```
            
- 코드 변환 과정
    - Java 코드(`HelloWorld.java`) → 컴파일 (`javac`) → 바이트코드 변환(`HelloWorld.class`) → JVM이 바이트코드를 실행 → JIT 컴파일러가 바이트코드를 기계어로 변환 → 기계어 실행(CPU가 직접 처리)

- 굳이 왜 바이트 코드?
    - Java의 핵심 철학은 **"Write Once, Run Anywhere" (한 번 작성하면 어디서든 실행할 수 있다)이다.**
    - 즉, **Windows, Mac, Linux 등 어떤 운영체제에서도 동일한 코드가 실행될 수 있어야 해**.
    
    👉 **그런데 기계어(어셈블리 코드)는 CPU나 OS마다 다름**
    👉 **바이트코드는 JVM 위에서 실행되므로, OS/CPU의 영향을 받지 않음**