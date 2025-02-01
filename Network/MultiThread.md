# 멀티 스레드

앞에서는 멀티 프로세스 기반의 서버 구현에 대해 살펴봤다.

멀티 프로세스 환경에서는 프로세스마다 완전히 독립된 메모리 공간을 유지하기 때문에 프로세스 사이에서 메세지를 주고 받아야하는 경우에는 그만클 구현의 어려움을 겪기도 한다. 즉, 멀티 프로세스 기반의 단점은 다음과 같다

<aside>
💡

"프로세스 생성이라는 부담스러운 작업과정을 거친다"

"두 프로세스 사이에서의 데이터 교환을 위해서는 별도의 IPC 기법을 적용해야한다"

</aside>

<br>

하지만 더 큰 단점이 존재한다.

😂 “초당 적게는 수십 번에서 많게는 수천 번까지 일어나는 '컨텍스트 스위칭(Context Switching)'에 따른 부담은 프로세스 생성방식의 큰 부담이다”

- CPU가 하나뿐인 시스템에서도 둘 이상의 프로세스가 동시에 실행된다.
- 이는 실행중인 둘 이상의 프로세스들이 CPU의 할당 시간을 매우 작은 크기로 쪼개서 서로 나누기 때문에 가능한 일이다.
- 그런데 CPU의 할당 시간을 나누기 위해서는 ‘Context Switch’ 과정을 거쳐야 한다.

결국 멀티 프로세스의 특징을 유지하면서 단점을 어느 정도 극복하기 위해서 Thread라는 것이 등장했는데, 이는 멀티프로세스의 여러 가지의 단점을 최소화하기 위해서(아주 없애는 것이 아니라), 설계된 일종의 '경량화 된(가벼워진) 프로세스'이다. 

쓰레드는 프로세스와 비교해서 다음의 장점을 가진다.

<br>

<aside>
💡

쓰레드의 생성 및 컨텍스트 스위칭은 프로세스의 생성 및 컨텍스트 스위칭보다 빠르다.

쓰레드 사이에서의 데이터 교환에는 특별한 기법이 필요치 않다.

</aside>

<br>

쓰레드는 다음과 같은 고민에 의해 등장하였다.

😵 "둘 이상의 실행 흐름을 갖기위해 프로세스가 유지하고 있는 메모리 영역을 통째로 복사하는 것이 부담스럽다!”

프로세스의 메모리 구조는 전역변수가 할당하는 '데이터 영역'

malloc 함수 등에 의해 동적 할당이 이뤄지는 '힙(Heap)'

그리고 함수의 실행에 사용되는 '스택(Stack)'으로 이뤄진다.

그런데 프로세스들은 이를 완전히 별도로 유지한다. 때문에 프로세스 사이에서는 다음의 메모리 구조를 보인다.

![image (6)](https://github.com/user-attachments/assets/3450b60b-2455-449e-83f5-9ab95de9efe7)


그런데 둘 이상의 실행흐름을 갖는게 목적이라면, 위 그림처럼 완전히 메모리 구조를 분리시킬것이 아니라, 스택 영역만을 분리시킴으로써 다음의 장점을 얻을 수 있다.

1. 컨텍스트 스위칭 시 데이터 영역과 힙은 올리고 내릴 필요가 없다.
2. 데이터 영역과 힙을 이용해서 데이터를 교환할 수 있다.

그래서 등장한 것이 쓰레드이며, 지금 설명한 것처럼 모든 쓰레드는 별도의 실행흐름을 유지하기 위해서 스택 영역만 독립적으로 유지하기 때문에 다음의 메모리 구조를 보인다.

![image (7)](https://github.com/user-attachments/assets/65fb73e0-5433-47dd-a2ec-c074a3192bc8)


쓰레드는 위 그림에서 보이듯이 데이터 영역과 힙 영역을 공유하는 구조로 설계되어 있다. 

그리고 이를 위해 쓰레드는 프로세스 내에서 생성 및 실행되는 구조로 완성되었다. 

즉, 프로세스와 쓰레드는 다음과 같이 정의할 수 있다.

- 프로세스 : `운영체제 관점`에서 별도의 실행흐름을 구성하는 단위
- 쓰레드 : `프로세스 관점`에서 별도의 실행흐름을 구성하는 단위

즉, 프로세스가 하나의 운영체제 안에서 둘 이상의 실행흐름을 형성하기 위한 도구라면, 쓰레드는 하나의 프로세스 내에서 둘 이상의 실행흐름을 형성하기 위한 도구로 이해할 수 있다. 

때문에 운영체제와 프로세스, 쓰레드의 관계는 다음과 같이 표현가능하다.

![image (8)](https://github.com/user-attachments/assets/b572c13c-22eb-4f54-b234-14bdc21e0b94)

<br>

### **스레드의 생성 및 실행**

`POSIX`란 Portable Operating System Inerface for Computer Environment의 약자로써 UNIX 계열 운영체제간에 이식성을 높이기 위한 표준 API 규격을 뜻한다. 

그리고 스레드의 생성방법은 POSIX에 정의된 표준을 근거로 한다. 때문에 리눅스 뿐만아니라 유닉스 계열의 운영체제에서도 대부분 적용 가능하다.

### **스레드의 생성과 실행흐름의 구성**

스레드는 별도의 실행흐름을 갖기 때문에 스레드만의 main 함수를 별도로 정의해야 한다. 

그리고 이 함수를 시작으로 별도의 실행 흐름을 형성해 줄 것을 운영체제에게 요청해야 하는데, 이를 목적을 호출하는 함수는 다음과 같다.

```c
#include <pthread.h>
int pthread_create(
    pthread_t *restict thread, const pthread_attr_t *restrict attr,
    void *(*start_routine)(void*), void *restrict arg
    );  // 성공시 0, 실패시 0 이외의 값 반환
```

- thread : 생성할 스레드의 ID 저장을 위한 변수의 주소 값 전달, 참고로 스레드는 프로세스와 마찬가지로 스레드의 구분을 위한 ID가 부여된다.
- attr : 스레드에 부여할 특성 정보의 전달을 위한 매개변수, NULL 전달 시 기본적인 특성의 스레드가 생성된다.
- start_routine : 스레드의 main 함수 역할을 하는, 별도 실행흐름의 시작이 되는 함수의 주소값(함수 포인터) 전달
- arg : 세번째 인자를 통해 등록된 함수가 호출될 때 전달할 인자의 정보를 담고 있는 변수의 주소 값 전달

```c
// thread1.c
#include <unistd.h>
#include <stdio.h>
#include <pthread.h>
void* thread_main(void *arg);

int main(int argc, char *argv[])
{
   pthread_t t_id;
   int thread_param = 5;

   if(pthread_create(&t_id, NULL, thread_main, (void*)&thread_param) != 0)
   {
      puts("pthread_create() error");
      return -1;
   };
   sleep(10);
   puts("end of main");
   return 0;
}

void *thread_main(void *arg)
{
   int i;
   int cnt = *((int*)arg);
   for(i = 0; i < cnt; i++)
   {
      sleep(1);
      puts("running thread");
   }
   return NULL;
}
```

![image (9)](https://github.com/user-attachments/assets/4c316136-b9ee-4890-beac-88330d70e960)


- 위 실행결과에서 보이듯이 쓰레드 관련 코드는 컴파일 시 -lpthread 옵션을 추가해서 쓰레드 라이브러리의 링크를 별도로 지시해야한다. 그래야 헤더파일 pthread.h에 선언된 함수들을 호출할 수 있다.
- 참고로 위 예제의 실행형태를 그림으로 표현하면 다음과 같다.

![image (10)](https://github.com/user-attachments/assets/b34b84f1-05b5-48c8-97f5-815ec965d337)


그럼 이번에는 예제 코드의 sleep함수의 호출문을 10에서 2로 바꿔서 실행해보자

![image (11)](https://github.com/user-attachments/assets/0539078f-4ddb-4d3e-b916-949e53219d45)


![image (12)](https://github.com/user-attachments/assets/eac33be1-b0f9-4e92-a1c5-2ce3500fccd1)


- sleep(2)로 바꿔서 실행하면 "running thread"의 출력이 5회 출력되지 않음을 볼 수 있다.
- 이는 다음 그림에서 보이듯이, main 함수의 종료로인해 프로세스 전체가 소멸되었기 때문이다.

따라서 위 예제는 sleep 함수 호출을 통해 쓰레드가 실행되기에 넉넉한 시간을 확보하고 있다.

하지만  쓰레드 기반 프로그래밍에는 적절한 sleep 함수 호출이 필수라고 생각할 수도 있다.

이는 그렇지 않은데, **sleep 함수의 호출을 통해 쓰레드의 실행을 관리**한다는 것은 프로그램의 흐름을 예측한다는 뜻인데, 이는 **사실상 불가능한 일**이다. 그리고 잘못된 구현은 프로그램의 흐름을 방해하는 결과로 이어질 수 있다.

이러한 문제점 때문에 sleep 함수보다는 다음 함수를 이용해서 쓰레드의 실행흐름을 조절할 수 있다. 즉, 다음 함수를 사용하면 지금 이야기하고 있는 문제를 쉽고 효율적으로 해결할 수 있다.

참고로 이 함수를 통해 쓰레드의 ID가 어떤 경우에 사용되는지도 함께 확인할 수 있다.

```c
#include <pthread.h>
int pthread_join(pthread_t thread, void **status); // 성공 시 0, 실패시 0 이외의 값 반환
```

- thread : 이 매개변수에 전달되는 ID의 쓰레드가 종료될 때까지 함수는 반환하지 않는다.
- status : 쓰레드의 main 함수가 반환하는 값이 저장될 포인터 변수의 주소 값을 전달한다.

간단히 말해서, 위 함수는 첫번째 인자로 전달되는 ID의 **쓰레드가 종료될 때까지, 이 함수를 호출한 프로세스(또는 쓰레드)를 대기 상태에 둔다**. 

뿐만 아니라, 쓰레드의 main 함수가 반환하는 값까지 얻을 수 있으니, 그만큼 유용한 함수이다. 다음 예제를 통해 함수의 기능을 확인해보자.

```c
// thread2.c
include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>
void* thread_main(void *arg);

int main(int argc, char *argv[])
{
   pthread_t t_id;
   int thread_param = 5;
   void * thr_ret;

   if(pthread_create(&t_id, NULL, thread_main, (void*)&thread_param) != 0)
   {
      puts("pthread_create() error");
      return -1;
   };

   if(pthread_join(t_id, &thr_ret) != 0)
   {
      puts("pthread_join() error");
      return -1;
   };

   printf("Thread return message : %s \n", (char*)thr_ret);
   free(thr_ret);
   return 0;
}

void *thread_main(void *arg)
{
   int i;
   int cnt = *((int*)arg);
   char * msg = (char*)malloc(sizeof(char)*50);
   strcpy(msg, "Hello, I'am thread~ \n");
   for(i = 0; i < cnt; i++)
   {
      sleep(1);
      puts("running thread");
   }
   return (void*)msg;
}
```

![image (13)](https://github.com/user-attachments/assets/3951a2da-d8ce-4504-8267-bdcd25cb1868)


![image (14)](https://github.com/user-attachments/assets/2e96a571-70ac-4a8c-880d-0c42da1799ca)


- 예제의 이해를 돕기위해 실행흐름을 그림으로 정리하면 다음 그림과 같다.
- 이 그림에서 주목할 부분은 실행이 일시적으로 정지되었다가 쓰레드가 종료되면서(쓰레드의 main 함수가 반환하면서) 다시 실행이 이어지고 있는 부분이다.


<br>

## 임계영역 내에서 호출이 가능한 함수

앞서 보인 예제에서는 쓰레드를 하나만 생성했었다. 그러나 이제부터는 둘 이상의 쓰레드를 생성해 볼  것이다.

물론 쓰레드를 하나 생성하건, 둘을 생성하건 그 방법에 있어서 차이를 보이진 않는다.

그러나 쓰레드의 실행과 관련해서 주의해야 할 사실이 하나있다.

함수 중에는 둘 이상의 쓰레드가 동시에 호출하면(실행하면) 문제를 일으키는 함수가 있다.

이는 함수 내에 '**임계영역(Critical Section)**'이라 불리는, 둘 이상의 쓰레드가 동시에 실행하면 문제를 일으키는 문장이 하나 이상 존재하는 함수이다.

어떠한 코드가 임계영역이 되는지, 그리고 둘 이상의 쓰레드가 동시에 임계영역을 실행하면 어떠한 문제가 발생하는지 잠시 후에 알아보도록 하고, 일단은 둘 이상의 쓰레드가 동시에 실행하면 문제를 일으키는 코드블록을 가리켜 임계영역이라 한다는 사실만 기억하자. 

이러한 임계영역의 문제와 관련해서 함수는 다음 두 가지 종류로 구분이 된다.

- 쓰레드에 안전한 함수(Thread-safe function)
- 쓰레드에 불안전한 함수(Thread-unsafe unction)

여기서 쓰레드에 안전한 함수는 둘 이상의 쓰레드에 의해 동시에 호출 및 실행되어도 문제를 일으키지 않는 함수이다.

반대로 쓰레드에 불안전한 함수는 동시 호출 시 문제가 발생할 수 있는 함수를 뜻한다. 하지만 이것이 임계영역의 유무를 뜻하는 것이 아니다.

즉, 쓰레드에 안전한 함수도 임계영역이 존재할 수 있다. 다만 이 영역을 둘 이상의 쓰레드가 동시에 접근해도 문제를 일으키지 않도록 적절한 조치가 이뤄져 있어 쓰레드에 안전한 함수로 구분될 수 있는 것이다.

다행히 기본적으로 제공되는 대부분의 표준함수들은 쓰레드에 안전하다. 

그러나 그보다 더 다행인 것은 쓰레드에 안전한 함수와 불안전한 함수의 구분을 우리가 직접할 필요는 없다는데 있다. 

왜냐하면 쓰레드의 불안전한 함수가 정의되어 있는 경우, 같은 기능을 갖는 쓰레드에 안전한 함수가 정의되어 있기 때문이다.

때문에 동일한 기능을 제공하면서 쓰레드에 안전한 다음 함수가 이미 정의되어 있다.

```c
struct hostent *gethostbyname_r(
        const char *name, struct hostent *result, char *buffer, intbuflen, int *h_errnop);
```

일반적으로 쓰레드에 안전한 형태로 재구현된 함수의 이름에는 _r이 붙는다.

그렇다면 둘 이상의 쓰레드가 동시에 접근가능한 코드블록에서는 gethostbyname 함수를 대신해서 gethostbyname_r을 호출해야하는 것은 아니다. 그런데 다행히도 다음의 방법으로 이를 자동화할 수 있다.

즉, 다음과 같은 방법을 통해 gethostbyname 함수의 호출문을 gethostbyname_r 함수의 호출문으로 자동으로 변경할 수 있다.

"헤더파일 선언 이전에 매크로 _REENTRANT를 정의한다."

gethostbyname 함수와 gethostbyname_r 함수가 이름에서 뿐만 아니라 매개변수의 선언에서도 차이가 난다는 사실을 알았다. 그리고 위의 매크로 정의를 위해 굳이 #define 문장을 추가할 필요는 없다.

다음과 같이 컴파일 시 -D_REENTRANT의 옵션을 추가하는 방식으로도 매크로를 정의할 수 있기 때문이다.

```arduino
gcc -D_REENTRANT mythread.c -o mthread -lpthread
```

따라서 앞으로 쓰레드 관련 코드가 삽입되어 있는 예제를 컴파일 할때는 -D_REENTRANT 옵션을 항상 추가해야한다.

## 워커 쓰레드 모델

지금까지는 쓰레드의 개념 및 생성 방법의 이해를 목적으로 아주 간단히 예제를 작성했기 때문에 하나으ㅢ 예제 안에서 둘 이상의 쓰레드를 생성해보지 못했다. 따라서 이번에는 둘 이사으이 쓰레드가 생성되는  예제를 작성해보고자 한다.

1~10까지 덧셈결과를 출력하는 예제를 만든다고 가정해보자.

그런데 main 함수에서 덧셈을 진행하는 것이 아니라, 두 개의 쓰레드를 생성해서 하나는 1~5까지, 다른 하나는 6~10까지 덧셈하도록하고, main 함수에서는 단지 연산결과를 출력하는 형태로 작성해 보고자 한다.

참고로 이러한 유형의 프로그래밍 모델을 가리켜 '워커 쓰레드(Worker thread) 모델' 이라고 한다.

1~5, 6~10 까지 덧셈을 진행하는 쓰레드가 main 함수가 관리하는 일꾼(Worker)의 형태를 띠기 때문이다.

그럼 마지막으로 예제를 작성하기 앞서 예제의 실행흐름을 그림으로 정리하면 다음과 같다.

![image (15)](https://github.com/user-attachments/assets/94a82399-6161-492d-b901-ae143567b236)


```c
// thread3.c
#include <stdio.h>
#include <pthread.h>
void * thread_summation(void * arg);
int sum = 0;

int main(int argc, char *argv[])
{
   pthread_t id_t1, id_t2;
   int range1[] = {1, 5};
   int range2[] = {6, 10};

   pthread_create(&id_t1, NULL, thread_summation, (void *)range1);
   pthread_create(&id_t2, NULL, thread_summation, (void *)range2);

   pthread_join(id_t1, NULL);
   pthread_join(id_t2, NULL);
   printf("result : %d \n", sum);
   return 0;
}

void * thread_summation(void * arg)
{
   int start = ((int*)arg)[0];
   int end = ((int*)arg)[1];

   while(start <= end)
   {
      sum += start;
      start++;
   }
   return NULL;
}
```

- 두 쓰레드가 하나의 전역변수 sum에 직접 접근하는 사실에는 별도의 주목이 필요하다.
- 위 예제 sum += start; 를 통해 이러한 결론을 내릴 수 있는데, 코드상에서 보면 이는 매우 당연한 것처럼 보인다.
    - 그러나 이는 전역변수가 저장되는 데이터 영역을 두 쓰레드가 함께 공유하기 때문에 가능한 것이다.

![image (16)](https://github.com/user-attachments/assets/8087b463-0511-4341-a884-401a46bf5227)


- 실행결과로써 55가 출력되었다.
- 물론 실행결과는 정확하지만 예제 자체적으로는 문제가 있다. 앞서 간단히 소개한 임계영역과 관련된 문제이다.

예제를 하나 더 작성해보자. 이 예제는 위의 예제와 거의 비슷하다.

다만, 앞서 설명한 임계영역과 관련해서 오류의 발생소지를 더 높였을 뿐이다. 이 정도 예제면 컴퓨터의 성능이 좋아도 어렵지 않게 오류의 발생을 확인할 수 있을 것이다.

```c
// thread4.c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>
#define NUM_THREAD 100

void * thread_inc(void * arg);
void * thread_des(void * arg);
long long num = 0;   // long long형은 64비트 정수 자료형

int main(int argc, char *argv[])
{
   pthread_t thread_id[NUM_THREAD];
   int i;

   printf("sizeof long long : %ld \n", sizeof(long long)); // long long 크기확인
   for(i = 0; i < NUM_THREAD; i++)
   {
      if(i % 2)
         pthread_create(&(thread_id[i]), NULL, thread_inc, NULL);
      else
         pthread_create(&(thread_id[i]), NULL, thread_des, NULL);
   }

   for(i = 0; i < NUM_THREAD; i++)
      pthread_join(thread_id[i], NULL);

   printf("result: %lld \n", num);
   return 0;
}

void * thread_inc(void *arg)
{
   int i;
   for(i = 0; i<50000000; i++)
      num += 1;
   return NULL;
}

void * thread_des(void * arg)
{
   int i;
   for( i = 0; i < 50000000; i++)
      num -= 1;
   return NULL;
}
```

- 위 예제에서는 총 100개의 쓰레드를 생성해서 그 중 반은 thread_inc를 쓰레드의 main함수로, 나머지 반은 thread_des를 쓰레드의 main 함수로 호출하고있다.
- 이로써 전역 변수 num에 저장된 값의 증가와 감소의 최종결과로 변수 num에는 0이 저장되어야한다. 그럼 실제로 0이 저장되는지 여러번 실행해보자.

![image (17)](https://github.com/user-attachments/assets/8706e68a-e802-45cf-9a1c-b797f70ded10)


- 놀랍게도 실행결과는 0이 아니다. 뿐만아니라 실행할때마다 매번 그 결과도 다르게 출력된다.
- 아직 이유까지는 모르겠으나, 어찌되었든 쓰레드를 활용하는데 있어서 큰 문제임이 틀림없다.

<br>

## 쓰레드의 문제점과 임계영역(Critical  Section)

### 둘 이상의 쓰레드 동시 접근 문제

- thread4.c의 문제점은 다음과 같다.

" 전역변수 num에 둘 이상의 쓰레드가 함께(동시에) 접근하고 있다. “

여기서 말하는 접근이란 주로 값의 변경을 뜻한다. 그런데 보다 다양한 상황에서 문제가 발생할 수 있기 때문에 문제의 원인이 무엇인지 정확히 이해해야 한다. 

그리고 예제에서는 접근의 대상이 전역변수였지만, 이는 전역변수였기 때문에 발생한 문제가 아니다. 어떠한 메모리 공간이라도 동시에 접근하면 문제가 발생할 수 있다.

하나의 예를 통해 동시 접근이 무엇인지, 왜 문제가 되는지 알아보자.

먼저 변수에 저장된 값을 1씩 증가시키는 연산을 두 개의 쓰레드가 진행하려는 상황이라고 가정해보자.

- 정상적인 접근의 예


![image (18)](https://github.com/user-attachments/assets/f2078532-b4b3-4f9a-8c5c-4b528133c710)

주목해야하는 사실이 하나 있는데, 그것은 값의 증가방식이다.

(왼쪽그림)

- 값의 증가는 CPU를 통한 연산이 필요한 작업이다.
- 따라서 그냥 변수 num에 저장된 값이 변수 num에 저장된 상태로 증가하지는 않는다. 이 변수에 저장된 값은 **thread1에 의해 우선 참조가 된다.**
- 그리고 thread1은 이 값을 CPU에 전달해서 1이 증가된 값 100을 얻는다. 마지막으로 연산이 완료된 값을 변수 num에 저장한다.
- 이렇게 해서 변수 num에 100이 저장되는 것이다.

(오른쪽그림)

- 왼쪽그림에 이어서 thread2도 값을 증가시키도록 되있는데, 이렇게해서 변수 num에는 101이 저장된다.

그런데 이는 매우 이상적인 상황을 묘사한 것이다. 위 그림처럼 **thread1, thread2에 순차적으로 변수 num에 접근하면 문제가 발생하지 않는다**.

**thread1이 변수 num에 저장된 값을 완전히 증가시키기 전에라도 얼마든지 thread2로 CPU의 실행이 넘어갈 수 있기 때문이다.**

그럼 처음부터 다시 시작해보자.

- 다음 그림의 왼쪽 부분은 thread1이 num에 저장된 값을 참조해서 값을 1증가시키는 것까지 완료한 상황을 보여준다.
- 단, 변수 num에는 아직 증가된 값을 저장하지 않았다.
- 이제 100이라는 값을 변수 num에 저장해야하는데 이 작업이 진행되기 전에 thread2로 실행의 순서가 넘어가 버렸다.
- 그런데 thread2는 증가연산을 완전히 완료해서, 증가된 값을 num에 저장했다고 가정해보자.

- 잘못된 접근의 예시 : thread1과 thread2가 각각 1씩 증가시켰는데, 변수 num의 값은 1만 증가하였다.

![image (19)](https://github.com/user-attachments/assets/ae146d43-3e3a-4d0d-9d5f-0e11190afdda)


(가운데 그림)

- 가운데 그림에서 보이듯이, 변수 num에 저장된 값이 thread1에 의해 100으로 증가된 상태가 아니기 때문에, thread2가 참조한 변수 num의 값은 99이다.
- 결국 thread2에 의해 변수 num의 값은 100이 되었다. 이제 남은 일은 thread1이 증가시킨 값을 변수 num에 저장하는 일만 남았다.

(오른쪽 그림)

- 안타깝게도 이미 100으로 증가된 변수 num에 다시 100을 저장하는일이 발생하였다.
- 결과적으로 num은 100이 된다. 비록 thread1과 thread2이 각각 1씩 증가시켰지만, 이렇게 전혀 엉뚱한 값이 저장될 수도 있는 것이다.
- 문에 이러한 문제를 막기위해 한 쓰레드가 변수 num에 접근해서 연산을 완료할때까지, 다른 쓰레드가 변수 num에 접근하지 못하도록 막아야한다. 바로 이것이 '동기화(Synchronization)'이다.

이제 멀티쓰레드 프로그래밍에서 동기화가 왜 필요한지 충분히 이해했을 것이다.

## **임계 영역은 어디?**

임계영억의 구분은 어렵지 않다. 앞에서는 임계영역을 다음과 같이 정의하였으니, 예제 thread4.c에서 임계영역을 찾아보자.

"함수 내에 둘 이상의 쓰레드가 동시에 실행하면 문제를 일으키는 하나 이상의 문장으로 묶여있는 코드 블록"

전역 변수 num을 임계영역으로 보는 건 아니다. 이는 문제를 일으키는 문장이 아니다.

뿐만아니라 동시에 실행이 되는 문장도 아닌, 메모리의 할당을 요구하는 변수의 선언일 뿐이다. 일반적으로 임계영역은 쓰레드에 의해 실행되는 함수 내에 존재한다. 예제 thread4.c에서 보인 쓰레드의 두 main 함수를 살펴보자.

```c
void * thread_inc(void *arg)
{
   int i;
   for(i = 0; i<50000000; i++)
      num += 1;     // 임계영역
   return NULL;
}

void * thread_des(void * arg)
{
   int i;
   for( i = 0; i < 50000000; i++)
      num -= 1;     // 임계영역
   return NULL;
}
```

위 코드의 주석에서 말하듯이 변수 num이 아닌, **변수 num에 접근하는 두 문장이 임계영역에 해당**한다.

이 두 문장은 둘 이상의 쓰레드에 의해 동시에 실행되도록 구현되어 있는, 문제를 일으키는 직접적인 원인이 되기 때문이다. 물론 문제가 발생하는 상황은 다음과 같이 세 가지 형태로 나눠서 정리할 수 있다.

- 두 쓰레드가 동시에 thread_inc 함수를 실행하는 경우
- 두 쓰레드가 동시에 thread_des 함수를 실행하는 경우
- 두 쓰레드가 각각 thread_inc 함수와 thread_des 함수를 동시에 실행하는 경우

여기서 마지막에 언급한 경우를 주목할 필요가 있다. 이는 다음의 경우에도 문제가 발생할 수 있음을 의미한다.

"쓰레드 1이 thread_inc 함수 num+=1을 실행할때,

동시에 쓰레드 2가 thread_des 함수의 문장 num-=1을 실행하는 상황"

이렇듯 임계영역은 서로 다른 두 문장이 각각 다른 쓰레드에 의해 동시에 실행되는 상황에서도 만들어질 수 있다.

바로 그 두 문장이 동일한 메모리 공간에 접근한다는 가정하에 말이다.
