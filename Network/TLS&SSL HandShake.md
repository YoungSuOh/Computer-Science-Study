# TLS/SSL HandShake

HTTPS에서 클라이언트와 서버 간 통신 전 SSL 인증서로 신뢰성 여부를 판단하기 위해 연결하는 방식

## 🔄 **2. TLS/SSL Handshake 과정**

![image.png](attachment:a626f59f-093b-4598-9856-db23ccf9d195:image.png)

1. Client Hello
    1. 클라이언트 → 서버
    2. 클라이언트는 지원하는 암호화 방식(Cipher Suite), TLS 버전, 랜덤 값(Client Random) 등을 보냄
2. Server Hello
    1. 서버 → 클라이언트
    2. 서버는 클라이언트가 제안한 암호화 방식 중 하나를 선택하고, 서버의 랜덤 값을 보냄
    3. 서버는 SSL/TLS 인증서(공개키도 포함)도 함께 전달
3. 인증서 검증 (Certificate Validation)
    1. 클라이언트는 서버의 인증서를 검증
    2. 신뢰할 수 있는 CA에서 발급되었는지 확인
        1. 유효하지 않으면 연결 종료
4. 키 교환  (Key Exchange)
    1. RSA 방식 : 클라이언트가 세션 키를 생성하고, 서버의 공개키로 암호화여 전
5. Finished
    1. **공유된 세션 키**를 사용하여 메시지를 암호화
    2. 클라이언트와 서버는 암호화된 "Finished" 메시지를 서로 전송하여 핸드셰이크 종료
    3. 이후부터 **모든 데이터는 암호화된 세션을 통해 전송**됨