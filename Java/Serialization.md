# Serialization

- **직렬화(Serialization)**: 자바 객체 → **바이트 스트림**으로 변환하여 저장 또는 전송 가능하게 만드는 과정
- **역직렬화(Deserialization)**: **바이트 스트림** → 원래의 **자바 객체**로 복원하는 과정

각자 PC의 OS마다 서로 다른 가상 메모리 주소 공간을 갖기 때문에, Reference Type의 데이터들은 인스턴스를 전달 할 수 없다.

따라서, 이런 문제를 해결하기 위해선 주소값이 아닌 Byte 형태로 직렬화된 객체 데이터를 전달해야 한다.

### 🔹 **직렬화가 필요한 이유**

1. **서로 다른 시스템 간 객체 전송 가능**
    - 운영체제(OS)마다 **메모리 주소 체계가 다름** → 같은 객체라도 메모리 주소가 다를 수 있음
    - 따라서, **객체를 바이트 스트림(Primitive Type)으로 변환하면** 메모리 주소와 관계없이 데이터를 주고받을 수 있음.
2. **파일 저장 및 네트워크 전송 가능**
    - 자바 객체는 메모리에만 존재하기 때문에, 파일로 저장하거나 네트워크로 전송하려면 **바이트 단위로 변환**해야 함.
3. **분산 시스템에서 객체 공유 가능**
    - Java RMI(Remote Method Invocation) 같은 기술을 사용할 때, **객체를 다른 JVM에서 사용할 수 있도록 직렬화해야 함**.

![image.png](attachment:687959de-ad82-42f1-b21c-9f2e52458030:image.png)

## 객체 직렬화

- 객체는 파일 또는 네트워크로 전송이 안된다.
- 객체를 byte[] 로 변환시켜서 전송해야 한다.
- 객체를 파일에 직렬화(serialize)하여 저장하고, 저장된 객체를 다시 역직렬화(deserialize)하여 읽어오기

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/78b2bb01-b78f-4ab2-9de7-c1e4834ec80a/Untitled.png)

- 직렬화 사용처
    - 휘발성이 있는 캐싱 데이터를 영구 저장이 필요할 때 사용할 수도 있다.
        - JVM의 메모리에서만 상주되어있는 객체 데이터가 시스템이 종료되더라도 나중에 다시 재사용이 될 수 있을 때 영속화 해두면 좋다.
        - ex) 서블릿 세션, 캐시, 자바 RMI
- 직렬화 vs JSON
    - 자바 직렬화는 외부 파일이나 네트워크를 통해 클라이언트 간에 객체 데이터를 주고 받을 때 사용된다.
    - WHY 직렬화 instead of JSON, CSV??
        
        ![스크린샷(310).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/8b9cc925-f38b-4d46-bbd0-54e827f3a7ec/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(310).png)
        
    - 직렬화의 장점
        - 자바의 고유 기술인 만큼 자바 시스템에서 개발에 최적화되어 있다.
        - 자바의 광활한 레퍼런스 타입에 대해 제약없이 외부에 내보낼 수 있다.
            - 예를 들어 기본형(int, double, string) 타입이나 배열과 같은 타입들은 왠만한 프로그래밍 언어가 공통적으로 사용하는 타입이기 때문에, 이러한 값들을 JSON으로도 충분히 이용 가능
            - 하지만 Wrapper 클래스, 인터페이스 타입들은 별도의 parsing이 존재
- 자바 직렬화 해보기
    
    ```java
    class Customer implements Serializable/*직렬화를 하겠다는 의도*/{
        int id;
        String name;
        String password;
        int age;
    
        public Customer(int id, String name, String password, int age) {
            this.id = id;
            this.name = name;
            this.password = password;
            this.age = age;
        }
        @Override
        public String toString() {
            return "Customer [id=" + id + ", name=" + name + ", password=" + password + ", age=" + age
                    +"]";
        }
    
    }
    
    ```
    
    ```java
    public class SerializeMain {
        public static void main(String[] args) {
            Customer customer = new Customer(1, "John", "Doe", 30);
    
            String fileName = "Customer.ser";
    
            try(FileOutputStream fos = new FileOutputStream(fileName);
                ObjectOutputStream oos = new ObjectOutputStream(fos)){
                oos.writeObject(customer);
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
    
    ```
    
    - 위를 실행하면 다음과 같은 내용의 Customer.ser 파일이 나온다.
    
    <aside>
    💡    sr serialize_.Customer94  e I ageI idL namet Ljava/lang/String;L passwordq ~ xp      t Johnt Doe
    
    </aside>
    
    ![스크린샷(311).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/9576ced0-6ff5-4af9-8109-250df3a7a4a6/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(311).png)
    
- 자바 역직렬화 하기
    
    ```java
    public class SerializeMain {
        public static void main(String[] args) {
            // 외부 파일명
            String fileName = "Customer.ser";
    
            // 파일 스트림 객체 생성 (try with resource)
            try(
                    FileInputStream fis = new FileInputStream(fileName);
                    ObjectInputStream in = new ObjectInputStream(fis)
            ) {
                // 바이트 스트림을 다시 자바 객체로 변환 (이때 캐스팅이 필요)
                Customer deserializedCustomer = (Customer) in.readObject();
                System.out.println(deserializedCustomer);
    
            } catch (IOException | ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
    }
    ```
    
    ![스크린샷(312).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/e4ca210a-3795-403d-9567-7da2e954a0be/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(312).png)
    

## **요약**

- 데이터를 통신 상에서 전송 및 저장하기 위해 직렬화/역직렬화를 사용한다.
- `serialVersionUID`는 개발자가 직접 관리한다.
- 클래스 변경을 개발자가 예측할 수 없을 때는 직렬화 사용을 지양한다.
- 개발자가 직접 컨트롤 할 수 없는 클래스(라이브러리 등)는 직렬화 사용을 지양한다.
- 자주 변경되는 클래스는 직렬화 사용을 지양한다.
- 역직렬화에 실패하는 상황에 대한 예외처리는 필수로 구현한다.
- 직렬화 데이터는 타입, 클래스 메타정보를 포함하므로 사이즈가 크다. 트래픽에 따라 비용 증가 문제가 발생할 수 있기 때문에 JSON 포맷으로 변경하는 것이 좋다.
    
    > JSON 포맷이 직렬화 데이터 포맷보다 2~10배 더 효율적
    >