# TCP (3 Way Handshake) + MultiProcess

## TCP 통신이란?

- 네트워크에서 신뢰적인 연결방식
- TCP는 기본적으로 unreliable network에서, reliable network를 보장할 수 있도록 하는 프로토콜

<br>

## 데이터의 송신 과정

![Untitled](https://github.com/user-attachments/assets/1c817971-b57f-469f-aba8-095a30d2d814)


송신 과정, naver D2 - TCP/IP 네트워크 스택 이해하기

<aside>
📨 **데이터 송신의 과정**

1. Application (데이터를 송신 하려는 서버 프로세스)
2. Sockets
3. 네트워크 스택
4. NIC
</aside>

1 ~ 4 의 각각의 계층 간 데이터의 이동이 정확히 어떻게 이루어 지는지는 몰라도 괜찮고,
간단히 **다양한 운영체제의 시스템 콜, 그리고 인터럽트를 활용한다** 정도만 이해해도 된다.

<br>

### 데이터를 송신할 때

서버 프로세스가 운영체제의 write 시스템 콜을 통해 소켓에 데이터를 보내게 되고, 이후 TCP / UDP 계층과 IP 계층, 그리고 대표적으로 Ethernet을 거쳐  흐름제어, 라우팅 등의 작업을 하게 된다.

이후, 마지막으로 NIC(랜 카드)를 통해 외부 데이터를 보낸다.

<br>

### 데이터를 수신할 때

데이터 수신 시에는 반대로 NIC에서 데이터를 수신하고, 인터럽트를 통해 Driver로 데이터를 옮기고 이후 네트워크 스택에서 데이터가 이동하며 소켓에 데이터가 담기고, 최종적으로 수신 대상이 되어 프로세스에 데이터가 도달하게 된다.

<br>

### Socket

**TCP 전용 소켓 (=stream 소켓)**

TCP는 UDP와 달리 신뢰성 있는 데이터 송수신을 하며 TCP 소켓을 활용하는 시스템 콜에서 이런 

특징이 드러나게 된다.

**UDP 소켓(=datagram 소켓)**

UDP는 TCP와 달리 비연결지향이다.

![Untitled (1)](https://github.com/user-attachments/assets/2832187a-7ba7-47a7-8cbc-cb0907682a11)


TCP 소켓을 사용하는 흐름

1. socket() 시스템 콜
    
    <aside>
    ✉️ ***socket(domain, type, protocol );***
    
    ---
    
    ***domain*** : IPv4, IPv6중 무엇을 사용할지 결정
    
    ***type*** : stream, datagram 소켓 중 선택
    
    ***protocol*** : 0, 6, 17 중 0을 넣으면 시스템이 프로토콜을 선택하며, 6이면 tcp, 17이면 udp
    
    </aside>
    
    - 소켓을 만드는 시스템 콜, 미리 형태를 잡아두는 것이라고 이해하면 됨.
        - 미리 Ipv4 통신을 위해 사용할지, Ipv6 통신을 위해 사용할지, TCP를 사용할지 아니면 UDP를 사용할지 **틀을 만들어두는 것**
    
    ```c
    int socket_descriptor;
    socket_descriptor = socket(AF_INET, SOCK_STREAM, 0);
    ```
    
    - socket()의 리턴 값은 `File Descriptor`이다.
        - `File Descriptor` : 운영체제에서 열린 파일을 식별하는 정수 값(핸들)
        - 즉, 프로그램이 파일을 읽거나 쓸 때 파일을 가리키는 식별자
    - 리눅스에서는 모든 것을 파일로 취급하며 Socket 역시 파일로 취급함
        - web server 프로세스가 데이터를 전송하기 위해 write(), read() 시스템 콜을 사용 할 때,
        대상 파일의 파일 디스크립터를 파라미터로 전송하여 OS에게 어떤 파일에 데이터를 작성할지, 혹은 어떤 파일의 데이터를 요청할지 결정한다.
        - 이때, 파일 디스크립터가 소켓의 파일 디스크립터인 경우, **소켓에 데이터를 작성(데이터 송신)** 혹은 **소켓의 데이터를 읽어들이는(데이터 수신) 동작**을 하게 되는 것이다.
     
<br>

2. bind() 시스템 콜
    
    <aside>
    ✉️ ***bind(sockfd, sockaddr, socklen_t)***
    
    ---
    
    ***sockfd***: 바인딩을 할 소켓의 파일 디스크립터
    
    ***sockaddr***: 소켓에 바인딩 할 아이피 주소, 포트번호를 담은 구조체
    
    ***socklen_t :*** 위 구조체의 메모리 크기
    
    </aside>
    
    ```c
    #include <sys/socket.h>
    #include <netinet/in.h>
    
    int main() {
        int sockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (sockfd == -1) {
            perror("Socket creation failed");
            return 1;
        }
    
        struct sockaddr_in server_address;
        server_address.sin_family = AF_INET;         // IPv4 주소 체계
        server_address.sin_addr.s_addr = INADDR_ANY; // 모든 가능한 IP 주소
        server_address.sin_port = htons(80);       // 포트 번호 80
    
        if (bind(sockfd, (struct sockaddr *)&server_address, sizeof(server_address)) == -1) {
            perror("Bind failed");
            return 1;
        }
    
        // 바인딩 성공 처리 및 작업 수행
    
        return 0;
    }
    ```
    
    - bind()  시스템 콜은 **생성한 소켓에 실제 아이피 주소와 포트 번호를 부여하는 시스템 콜**
    - OS에게 **어떤 소켓에 아이피 주소와 포트번호 부여할지 알려주기 위해** 파라미터에 소켓의 파일디스크립터를 포함한다.
    - 클라이언트는 통신 시 포트번호가 자동으로 부여되기에 bind 시스템 콜은 서버에만 사용한다.
  
<br>

3. listen() (⭐only for TCP / 중요)
    
    <aside>
    ✉️ ***listen(sockfd, backlog)***
    
    ---
    
    ***sockfd :*** 소켓의 파일 디스크립터
    
    ***backlog :***  연결요청을 받아줄 크기 = TCP의 백로그 큐의 크기
    
    </aside>
    
    ```c
    #include <sys/socket.h>
    
    int main() {
        int sockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (sockfd == -1) {
            perror("Socket creation failed");
            return 1;
        }
    
        // ... 서버 소켓의 주소와 바인딩 설정 ...
    
        int backlog = 10; // 최대 대기열 크기
        if (listen(sockfd, backlog) == -1) {
            perror("Listen failed");
            return 1;
        }
    
        // 리스닝 성공 처리 및 연결 요청 처리
    
        return 0;
    }
    ```
    
    - 파라미터로 받은 File Descriptor에 해당하는 소켓을 클라이언트의 요결 요청을 받아들이도록 하며, 최대로 받아주는 크기를 backlog로 설정
    - 이 **listen() 시스템 콜에서 설정하는 backlog**가 **TCP에서의 backlog queue의 크기**입니다
    
    ### 🤔 그러면 어떻게 연결 ****요청을 받아들이게 하는 걸까? 🤔
    
    ![Untitled (2)](https://github.com/user-attachments/assets/c63d1911-947d-4e0d-a13c-d6310d334485)

    
    - 서버 측의 listen() 이후 대기 상태에서 클라이언트의 연결 요청을 받아주기 위해 backlog queue를 가진 채로 기다리게 된다.
    - 실제로는 서버에 셀 수 없이 많은 클라이언트가 요청을 보내게 되고, 이 요청들은 모두 backlog queue에 저장이 된다.
    
    그렇다면 큐에서 대기 중인 요청은 어떻게 처리가 될까용??
    
    - 뒤에 `accept() 시스템 콜`의 동작을  보면 이해가 될 것이다.
  
<br>

4. accept() 시스템 콜 ( ⭐⭐매우 놀랍도록 중요)
    
    <aside>
    ✉️ ***int accept(sockfd, sockaddr , socklen_t);***
    
    ---
    
    ***sockfd :*** 백로그 큐의 요청을 받아들이기 위한 소켓의 파일 디스크립터
    
    ***sockaddr  :*** 선입선출로 빼온 연결 요청에서 알아낸 클라이언트의 주소 정보
    
    ***socklen_t :*** 위 구조체의 메모리 크기
    
    </aside>
    
    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <string.h>
    #include <sys/socket.h>
    #include <netinet/in.h>
    
    int main() {
    		// 1. 소켓 생성 (IPv4, TCP)
        int server_socket = socket(AF_INET, SOCK_STREAM, 0);
    
    		// 2. 서버 주소 구조체 설정
        struct sockaddr_in server_address;
        server_address.sin_family = AF_INET; // IPv4 사용
        server_address.sin_addr.s_addr = INADDR_ANY; // 모든 IP에서 접속 허용 (0.0.0.0)
        server_address.sin_port = htons(80);  // 포트 번호 80 (웹 서버 기본 포트)
    
    		 // 3. 소켓과 주소 바인딩
        bind(server_socket, (struct sockaddr *)&server_address, sizeof(server_address));
    
    		// 4. 연결 요청 대기 (백로그 큐 크기: 5)
        listen(server_socket, 5);
    
        printf("Server: Waiting for client's connection...\n");
    
    	
        struct sockaddr_in client_address;
        socklen_t client_addrlen = sizeof(client_address);
        
    		// 5. 클라이언트 연결 수락
        int client_socket = accept(server_socket, (struct sockaddr *)&client_address, &client_addrlen);
    
    		// 6. 클라이언트 정보 출력
        printf("Server: Accepted connection from %s:%d\n",
               inet_ntoa(client_address.sin_addr), ntohs(client_address.sin_port));
    
        // 7. 클라이언트의 ACK 수신 (3-Way Handshake 마지막 단계)
        char buffer[1024];
        ssize_t bytes_received = recv(client_socket, buffer, sizeof(buffer), 0); // 클라이언트의 ACK 받기
        if (bytes_received > 0) {
            printf("Server: Received ACK from client.\n");
        }
        
        // 8. 소켓 닫기
        close(client_socket);
        close(server_socket);
    
        return 0;
    }  
        
        
    ```
    
    - accept 시스템 콜은 backlog queue에서 ***syn***을 보내와 대기 중인 요청을 선입선출(자료구조 큐)로 하나씩 연결에 대한 수립을 해줍니다.
    - **파라미터를 보면 클라이언트의 아이피 주소, 포트번호를 받는데 이 값은 백로그 큐에서 가장 앞에 있는 연결 요청 구조체에서 알아내서 가져온다.**
    
    <aside>
    🤔 **accept() 시스템 콜의 리턴 값은 새로운 소켓의 파일 디스크립터인데,
    왜 새로운 소켓을 만들지?? 하는 생각이 드실 것 입니다.**
    
    **이 의문에 대한 답은 아래 내용을 계속 읽으면 나오게 됩니다.**
    
    **그러니 우선은 그러려니 하고 넘어갑시다.**
    
    </aside>

<br>
    
5. TCP  3 Way Handshake
    
    ![Untitled (3)](https://github.com/user-attachments/assets/8de98d9b-d0d8-4d8a-b1cb-57e95523cef3)
    
    - TCP 3-way handshake는 클라이언트와 서버간에 서로 신뢰성있는 통신을 위해 서로 준비가 되었다! 이거를 확인하는 과정이라고 이해하시면 된다.
    - 위의 3과정 중 client가 보내는 SYN이 listen 상태인 서버의 소켓에 연결을 요청을 보내는 것이라고 생각하시면 되고, 이후 2 과정 (SYN, ACK / ACK)은 accept 시스템 콜 이후 진행하여 최종적으로 **`Established 상태`** 를 수립하고 본격적인 데이터의 송/수신 이뤄집니다.
    - 실제 listen 이후 동작을 아래 그림처럼 표현이 가능하다.
    
    ![Untitled (4)](https://github.com/user-attachments/assets/0c226905-d799-41e9-9611-6e1f743c6431)

    
    - 당연하게도 위 그림 중 다음 과정도 OS가 제공하는 system call을 통해 이루어진다~ 이말이야
    
    ![Untitled (5)](https://github.com/user-attachments/assets/bf97f689-6dc3-4e41-8bf0-a8f591c7652b)


<br>    

### Accept 시스템 콜 이후 멀티 프로세스/멀티 쓰레드

사실 accept 시스템 콜 이후 곧바로 잔여 3-way handshake 이후 데이터 송/수신이 이루어지는 것은 아니다.

서버의 성능을 위해 또 하나의 테크닉이 들어가게 된다.

**해당 테크닉은 멀티 프로세스, 혹은 멀티 쓰레드이다.**

하나의 프로세스인 서버가 클라이언트의 수많은 요청을 받는 상황에서, backlog queue의 가장 앞에 있던 클라이언트의 요청을 받고 응답까지 다 주고 다시 다음 요청을 받아준다면

⇒ **엄청난 병목이 생길 것이다.**

⭐ **따라서 서버는 연결 요청을 받는 부분 따로, 이후 응답까지 주는 부분을 따로 나누게 된다!**

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main() {
    // 1. 서버 소켓 생성 (IPv4, TCP)
    int server_socket = socket(AF_INET, SOCK_STREAM, 0);

    // 2. 서버 주소 구조체 설정
    struct sockaddr_in server_address;
    server_address.sin_family = AF_INET;           // IPv4 사용
    server_address.sin_addr.s_addr = INADDR_ANY;   // 모든 IP에서 접속 허용 (0.0.0.0)
    server_address.sin_port = htons(80);           // 포트 번호 80 (웹 서버 기본 포트)

    // 3. 소켓과 주소 바인딩
    bind(server_socket, (struct sockaddr *)&server_address, sizeof(server_address));

    // 4. 클라이언트의 연결 요청을 대기 (백로그 큐 크기: 5)
    listen(server_socket, 5);
    printf("Server: Listening on port 80...\n");

    while (1) { // 여러 클라이언트의 요청을 계속 처리
        struct sockaddr_in client_address;
        socklen_t client_addrlen = sizeof(client_address);

        // 5. 클라이언트의 연결 요청 수락
        int client_socket = accept(server_socket, (struct sockaddr *)&client_address, &client_addrlen);

        if (fork() == 0) { // 🔹 새로운 프로세스를 생성하여 클라이언트 처리 (멀티프로세싱)
            printf("Server: Accepted connection from %s:%d\n",
                   inet_ntoa(client_address.sin_addr), ntohs(client_address.sin_port));

            // 🔹 클라이언트의 요청을 처리하는 코드 (여기서는 간단히 sleep으로 대체)
            sleep(1); 

            // 6. 클라이언트에게 HTTP 응답 전송
            char response[] = "HTTP/1.1 200 OK\r\nContent-Length: 13\r\n\r\nHello, world!";
            send(client_socket, response, strlen(response), 0);
            printf("Server: Sent response to client.\n");

            // 7. 클라이언트 소켓 종료
            close(client_socket);
            exit(0); // 🔹 자식 프로세스 종료
        }

        // 🔹 부모 프로세스에서는 클라이언트 소켓을 닫고 다음 요청을 기다림
        close(client_socket);
    }

    // 8. 서버 소켓 종료 (여기까지 도달할 일은 없지만 종료 코드 포함)
    close(server_socket);

    return 0;
}

```

<aside>
👉 ***fork() - 자식 프로세스 생성***

**리턴값 = 0 자식 프로세스, 0이 아님 = 원래 본인(부모) 프로세스**

</aside>

<aside>
📌 accept 시스템 콜의 응답을 받았다
= SYN 요청을 보낸 클라이언트가 적어도 하나 있어서 백로그 큐에 있었고 해당 클라이언트의 요청에 대한 이후 응답을 위해 **새로운 소켓을 만들었다.**

</aside>

### 👨‍👩‍👧‍👦 부모 프로세스의 입장

```c
#include <stdio.h#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main() {
    int server_socket = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in server_address;
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = INADDR_ANY;
    server_address.sin_port = htons(80); // 웹 서버 포트인 80

    bind(server_socket, (struct sockaddr *)&server_address, sizeof(server_address));

    listen(server_socket, 5);

    printf("Server: Listening on port 80...\n");

    while (1) {
        struct sockaddr_in client_address;
        socklen_t client_addrlen = sizeof(client_address);
				// 연결 요청 받아줌
        int client_socket = accept(server_socket, (struct sockaddr *)&client_address, &client_addrlen);

				// 자식 프로세스가 아니므로 fork() 가 안
        if (fork() == 0 -> false ) { 
           실행안함
        }

      
    }

    close(server_socket);

    return 0;
	}
```

위의 코드를 보면 사실상 부모 프로세스는 연결 요청을 받아주고,

자식 프로세스에게 나머지 일을 맡기고 **다시 새로운 연결요청을 받아주는 형태**인 것을 확인 할 수 있다.

<br>

### 👨‍👩‍👧‍👦 자식 프로세스의 입장

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main() {
    int server_socket = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in server_address;
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = INADDR_ANY;
    server_address.sin_port = htons(80); // 웹 서버 포트인 80

    bind(server_socket, (struct sockaddr *)&server_address, sizeof(server_address));

    listen(server_socket, 5);

    printf("Server: Listening on port 80...\n");

    while (1) {
        struct sockaddr_in client_address;
        socklen_t client_addrlen = sizeof(client_address);

        int client_socket = accept(server_socket, (struct sockaddr *)&client_address, &client_addrlen);

        if (fork() == 0 -> true) { // 자식 프로세스
            close(server_socket);

            printf("Server: Accepted connection from %s:%d\n",
                   inet_ntoa(client_address.sin_addr), ntohs(client_address.sin_port));

            // 3-way handshake의 나머지 두 단계 수행
            // 여기서는 ACK를 보내는 과정만 간단히 보여줍니다.
            sleep(1); // 실제로는 필요한 로직 수행

            // 서버의 응답 전송
            char response[] = "HTTP/1.1 200 OK\r\nContent-Length: 13\r\n\r\nHello, world!";
            send(client_socket, response, strlen(response), 0);
            printf("Server: Sent response to client.\n");

            close(client_socket);
            exit(0); <-여기서 자식 프로세스가 종료됨
        }

        close(client_socket);
    }

    close(server_socket);

    return 0;
}
```

자식 프로세스는 반면, 부모 프로세스가 새로 만들어준 소켓을 이어받아

이후 남은 **잔여 3-way handshake를 수행 후 데이터 통신을 수행**하는 것을 확인 할 수 있다.

```c
if (fork() == 0 -> true) { // 자식 프로세스
            close(server_socket);

            printf("Server: Accepted connection from %s:%d\n",
                   inet_ntoa(client_address.sin_addr), ntohs(client_address.sin_port));

            // 3-way handshake의 나머지 두 단계 수행
            // 여기서는 ACK를 보내는 과정만 간단히 보여줍니다.
            sleep(1); // 실제로는 필요한 로직 수행

            // 서버의 응답 전송
            char response[] = "HTTP/1.1 200 OK\r\nContent-Length: 13\r\n\r\nHello, world!";
            send(client_socket, response, strlen(response), 0);
            printf("Server: Sent response to client.\n");

            close(client_socket);
            exit(0); <-여기서 자식 프로세스가 종료됨
        }
```

이 코드를 다시 보면 **if(fork() == 0)이 true인 경우(자식 프로세스) 마지막에 exit(0) 시스템 콜을 호출** 하는 것을 확인 할 수 있다.

**이는 곧 자식 프로세스는 새로운 연결요청을 받지 않고 그저 응답을 준 후 exit(0)를 통해 종료된다!**

이후 시스템 콜, recv(), send()는 말 그대로 데이터를 수신, 송신 할 때 사용되는 시스템 콜 입니다.

<br>

### 😀 핵심 요약

<aside>
📢 **서버는 연결을 받는 부분과 응답을 주는 부분이 병렬적으로 이루어져 있다**

</aside>

멀티 쓰레드는 나중에 다룰 생각입니다. 왜냐면 **너무 어려워서…🥲 야발..**

### *fin.* HTTP 웹 서버 코드

이제 위에서 다룬 코드를 합쳐 우리에게 익숙한 아파치(HTTP 웹 서버)의 간략화된 코드를 봐봅시다!

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

int main() {
    const char* server_ip = "127.0.0.1";
    int server_port = 8080;

    int server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket == -1) {
        perror("Socket creation failed");
        return 1;
    }

    struct sockaddr_in server_addr, client_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(server_port);
    server_addr.sin_addr.s_addr = inet_addr(server_ip);

    if (bind(server_socket, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("Binding failed");
        return 1;
    }

    if (listen(server_socket, 5) == -1) {
        perror("Listening failed");
        return 1;
    }

    printf("Server listening on %s:%d\n", server_ip, server_port);

    while (1) {
        socklen_t client_addr_len = sizeof(client_addr);
        int client_socket = accept(server_socket, (struct sockaddr*)&client_addr, &client_addr_len);
        if (client_socket == -1) {
            perror("Accepting client failed");
            continue;
        }

        printf("Accepted connection from %s:%d\n", inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

        char request[1024];
        recv(client_socket, request, sizeof(request), 0);
        printf("Received request:\n%s\n", request);

        char response[] = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\nHello, World!";
        send(client_socket, response, sizeof(response), 0);

        close(client_socket);
    }

    close(server_socket);
    return 0;
}
```
