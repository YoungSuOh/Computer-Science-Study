## `execl()`

```c
int execl(const char *path, const char *arg, ..., NULL);
```

- `path`: 실행할 프로그램의 **절대 경로 또는 상대 경로를 지정**해야 함.
- `arg`: 실행할 프로그램의 첫 번째 인자로, 관례적으로 실행할 파일명을 전달함.
- `NULL`: 인자 리스트의 마지막을 나타내야 함.

## `execlp()`

```c
int execlp(const char *file, const char *arg, ..., NULL);
```

- `file`: 실행할 프로그램의 이름만 지정 가능 (환경 변수 `$PATH`에서 검색)
- `arg`: 실행할 프로그램의 첫 번째 인자로 파일명을 전달함.
- `NULL`: 인자 리스트의 마지막을 나타내야 함.

## 문제 1: `fork()`와 `wait()`을 활용한 부모-자식 프로세스

- 다음 코드를 실행했을 때 출력 순서를 예상하시오.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        printf("부모 프로세스: 자식이 끝날 때까지 기다림\n");
        wait(NULL);
        printf("부모 프로세스: 자식이 종료됨\n");
    } else if (pid == 0) {
        printf("자식 프로세스 실행 중...\n");
        sleep(2);
        printf("자식 프로세스 종료\n");
    } else {
        perror("fork 실패");
        return 1;
    }

    return 0;
}
```

- result
    
    ```
    부모 프로세스: 자식이 끝날 때까지 기다림
    자식 프로세스 실행 중...
    자식 프로세스 종료
    부모 프로세스: 자식이 종료됨
    ```
    
    - `wait(NULL);`이 없으면 부모 프로세스가 자식이 종료되기 전에 먼저 종료될 가능성이 있음. 그러면 부모가 종료되고 나서야 자식 프로세스가 계속 실행될 수도 있음. 이를 **"좀비 프로세스"** 문제라고 함.
    - `sleep(2);`을 제거하면 자식 프로세스의 실행이 더 빠르게 끝나서, 부모 프로세스가 `wait()`을 호출할 때 이미 자식 프로세스가 종료되었을 수도 있음.

## 문제 2: `execl()`을 활용한 새로운 프로그램 실행

- 다음 코드를 실행하면 어떤 결과가 나오는가?

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    printf("새로운 프로그램 실행 전\n");
    execl("/bin/ls", "ls", "-l", NULL);
    printf("이 메시지가 출력될까?\n");

    return 0;
}

```

- result
    
    ```
    새로운 프로그램 실행 전
    total 12
    -rwxr-xr-x  1 user user  1234 Mar 20 14:00 example
    -rwxr-xr-x  1 user user  4321 Mar 20 14:00 another_file
    ```
    
    - `"이 메시지가 출력될까?"` → **출력되지 않음**
        - `execl()`이 실행되면 현재 프로세스가 `ls`로 대체되므로, 이후의 `printf("이 메시지가 출력될까?\n");`는 실행되지 않음.
    - 부모 프로세스도 실행되도록 하려면 `fork()`를 사용해서 자식 프로세스에서 `execl()`을 실행해야 함:
        
        ```c
        pid_t pid = fork();
        if (pid == 0) {
            execl("/bin/ls", "ls", "-l", NULL);
        }
        ```
        

## 문제 3: `execlp()`를 활용한 명령 실행

- 다음 코드를 실행했을 때 예상되는 출력을 작성하시오.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        wait(NULL);
        printf("부모 프로세스: 자식이 실행 완료됨\n");
    } else if (pid == 0) {
        printf("자식 프로세스: 'ls' 실행\n");
        execlp("ls", "ls", "-a", NULL);
        perror("execlp 실패");
        exit(1);
    } else {
        perror("fork 실패");
        return 1;
    }

    return 0;
}
```

- result
    
    ```
    자식 프로세스: 'ls' 실행
    .  ..  file1  file2  file3
    부모 프로세스: 자식이 실행 완료됨
    ```
    
    - `execlp("ls", "ls", "-a", NULL);`는 현재 환경 변수의 `PATH`에서 `ls`를 찾아 실행함. 따라서 `/bin/ls`를 직접 지정하지 않아도 됨.
    - `perror("execlp 실패");`가 실행되는 경우
        - `ls` 실행 파일을 찾을 수 없는 경우 (`$PATH` 환경 변수가 올바르지 않거나 `ls`가 존재하지 않는 경우)
        - 실행 권한이 없는 경우

## 문제 4: `fork()`, `wait()`, `execl()`을 함께 활용한 프로세스 관리

- 아래 코드에서 자식 프로세스가 새로운 프로그램을 실행하도록 수정하시오.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        printf("부모 프로세스: 자식이 실행되길 기다림\n");
        wait(NULL);
        printf("부모 프로세스 종료\n");
    } else if (pid == 0) {
        printf("자식 프로세스: 새로운 프로그램 실행\n");
        execl("/bin/date", "date", NULL);
        perror("execl 실패");
        exit(1);
    } else {
        perror("fork 실패");
        return 1;
    }

    return 0;
}

```

- result
    
    ```
    부모 프로세스: 자식이 실행되길 기다림
    자식 프로세스: 새로운 프로그램 실행
    Wed Mar 20 14:30:00 UTC 2025
    부모 프로세스 종료
    ```
    
    - `execl("/bin/date", "date", NULL);` → `date` 명령어가 `/bin/` 경로에 있어야 함.
    - `execlp("date", "date", NULL);` → `PATH` 환경 변수를 검색하여 `date`를 실행함.

## 문제 5 : 부모 프로세스 & 자식 프로세스

```c
#include<sys/types.h>
#inlcude<stdio.h>
#include<unistd.h>

int value=5;

int main(){
	pid_t pid;
	pid=fork();
    if(pid==0) { /*child process */
    	value+=15;
        return 0;
        }
    else if(pid>0) { /* parent process */
    	wait(NULL);
        printf("PARENT: value =%d",value); /*LINE A */
        return 0;
        }
  }
```

- result
    - 새로운 프로세스는 fork() 시스템 호출로 생성되는데, 이때 새로운 프로세스는 원래 프로세스의 주소 공간의 복사본으로 구성이 된다.
    - **A라인은 부모 프로세스의 공간이므로 자식 프로세스의 value+=15가 적용되지 않는다.**
    - **따라서 A행의 출력은 "PARENT: value =5" 이다**

## 문제 6 : `fork()`의 중첩 호출

```c
#include<stdio.h>
#include<unistd.h>
int main()
{
	/* fork a child process */
    fork();
    /* fork another child process */
    fork();
    /* and fork another */
    fork();
    return 0;
}
```

- result
    - `fork()` 시스템 콜은 현재 메모리에 있는 프로세스를 복사하여 자식 프로세스를 생성하는 프로세스이다.
    1. 첫 번째 `fork()`
        1. 첫 번째 `fork()` 를 통해 메모리에 있는 프로세스는 2개가 되었다.
    2. 두 번째 `fork()`
        1. 두 번째 fork()에서 메모리에 있는 프로세스의 개수는 2개이므로, 복사본이 2개가 더 생겨 4개의 프로세스가 생긴다.
    3. 세 번째 `fork()`
        1. 세 번째 fork에서 메모리에 있는 프로세스의 개수는 4개이므로 복사본 4개가 더 생겨 총 8개의 프로세스가 생긴다.
    
    📌 정리
    
    - 쉽게 말하면  fork()는 자식 프로세스들을 생성하고 또 다시 fork를 한다면 전에 생성된 자식프로세스들도 자식을 낳는다.  즉 2의 3승 =2*2*2이 되는 것이다

## 문제 7 : `fork()`와 `if`문 조합 (난이도 높음)

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    fork();
    if (fork()) {
        fork();
    }
    printf("Running process: PID = %d\n", getpid());
    return 0;
}
```

- result
    1. 첫 번째 `fork()` 호출 → **A (부모)와 B (자식) 생성** (총 **2개 프로세스**)
    2. `if (fork())`에서 부모 프로세스에서만 `fork()` 호출 → **3개로 증가**
        - **A(부모)만** `fork()`를 실행 (자식 `C` 생성)
        - **B(자식)는 `fork()` 반환값이 0이므로 `if` 문 실행 안 함** (B는 여기서 멈춤)
    3. 부모 프로세스에서 추가적인 `fork()` 호출 → **총 5개 프로세스**
        1. **A만** 새로운 프로세스 `D`를 생성 (C에서 실행 X)
    4. 최종 트리
        
        ```mathematica
        A
        ├── B
        ├── C
        │   ├── D
        │   └── E
        ```
        

## 문제 8 : `fork()`와 `wait()` 조합

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    fork();  
    fork();
        
    if (fork()) {
        wait(NULL);
    }
    
    printf("Process running: PID = %d\n", getpid());
    return 0;
}
```

- result
    1. 첫 번째 `fork()` → **2개 프로세스 (부모 A, 자식 C1)**
    2. 두 번째 `fork()` → **4개 프로세스**
        1. 트리 구조
            
            ```mathematica
                   P
                  / \
                 C1  P2
                /
               C2        
            ```
            
    3. `if (fork())` → 부모 프로세스는 `fork()`를 한 번 더 실행하므로 **총 6개 프로세스**
        - P와
        
        ```mathematica
               P
              / \  \
             C1  P2 P3
            /  \    
           C2   C3  
                  
               
        ```
        
    4. `wait(NULL);` 호출된 프로세스는 자식이 종료될 때까지 대기함.