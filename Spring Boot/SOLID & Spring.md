# SOLID & Spring

## 단일 책임 원칙(Single Responsibility Principle/SRP)

"클래스는 단 하나의 책임만 가져야 한다.”

- Spring에서는 **Service, Repository, Controller를 분리**하여 각각의 역할을 담당하게 함.
- 예를 들어, **비즈니스 로직**은 `Service`에, **데이터 접근**은 `Repository`에, **요청 처리**는 `Controller`에 둠으로써 SRP를 적용할 수 있음.

## ⭐ 개방-폐쇄 원칙(Open/Closed Principle/OCP)

"확장에는 열려 있고, 변경에는 닫혀 있어야 한다.”

- Spring의 **DI(Dependency Injection)와 Bean 설정**을 활용하면 코드 변경 없이 새로운 기능을 추가할 수 있음.
- 예를 들어, 인터페이스를 사용하면 기존 코드를 수정하지 않고 새로운 구현체를 추가 가능.

```java
public interface PaymentService {
    void pay();
}

@Component
public class CreditCardPaymentService implements PaymentService {
    public void pay() {
        System.out.println("신용카드로 결제");
    }
}

@Component
public class PaypalPaymentService implements PaymentService {
    public void pay() {
        System.out.println("PayPal로 결제");
    }
}

@Service
public class OrderService {
    private final PaymentService paymentService;

     @Autowired
    public OrderService(@Qualifier("paypalPaymentService") PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void processOrder() {
        paymentService.pay();
    }
}
```

- `PaymentService` 인터페이스를 사용하여 `OrderService`의 코드를 수정하지 않고 결제 방식을 확장할 수 있음
- `@Qualifier("paypalPaymentService")`를 사용하여 `PaypalPaymentService`를 명확히 지정.

## 리스코프 치환 원칙(Liskov Substitution Principle/LSP)

"하위 타입은 상위 타입을 대체할 수 있어야 한다.”

- 프로그램의 객체는 프로그램의 **정확성을 깨뜨리지 않으면서** 하위 타입의 인스턴스로 바꿀 수 있어야 한다.
    - ex) 자동차 인터페이스의 엑셀은 앞으로 가는 기능
- Spring의 **DI와 다형성**을 활용하면, 상위 타입(인터페이스)을 사용하여 하위 클래스가 문제없이 교체될 수 있도록 보장함.
- 예를 들어, `JdbcUserRepository`, `JpaUserRepository` 같은 여러 `UserRepository` 구현체를 상위 인터페이스(`UserRepository`)를 통해 교체 가능.

```java
public interface NotificationService {
    void sendNotification(String message);
}

@Component
public class EmailNotificationService implements NotificationService {
    public void sendNotification(String message) {
        System.out.println("이메일 전송: " + message);
    }
}

@Component
public class SmsNotificationService implements NotificationService {
    public void sendNotification(String message) {
        System.out.println("SMS 전송: " + message);
    }
}

@Service
public class AlertService {
    private final NotificationService notificationService;

    @Autowired
    public AlertService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void alert(String message) {
        notificationService.sendNotification(message);
    }
}
```

- `AlertService`는 `NotificationService` 인터페이스를 사용하기 때문에 이메일에서 SMS로 쉽게 변경 가능.

## 인터페이스 분리 원칙(Interface Segregation Principle/ISP)

"클라이언트가 사용하지 않는 기능에 의존하지 않도록 인터페이스를 분리해야 한다.”

- 하나의 거대한 인터페이스보다는 **작고 역할이 명확한 인터페이스**를 여러 개 만드는 것이 좋음.
- Spring의 **Service, Repository 인터페이스 분리**가 좋은 예시.

### 🙅 **잘못된 예시 (ISP 위반 - 하나의 인터페이스에 모든 기능 포함)**

```java
java
코드 복사
public interface UserService {
    void registerUser(User user);
    void deleteUser(Long userId);
    User findUserById(Long userId);
    List<User> findAllUsers();
    void changeUserPassword(Long userId, String newPassword);
}

```

- 회원 가입, 삭제, 조회, 비밀번호 변경까지 **하나의 인터페이스에 모든 기능을 포함**하고 있음.
- 일부 구현체는 필요 없는 기능까지 구현해야 하는 불필요한 의존성이 생김.

### **✅ 올바른 예시 (ISP 적용 - 인터페이스 분리)**

```java
// 읽기 전용 인터페이스
public interface UserReadService {
    User findUserById(Long userId);
    List<User> findAllUsers();
}

// 쓰기 전용 인터페이스
public interface UserWriteService {
    void registerUser(User user);
    void deleteUser(Long userId);
    void changeUserPassword(Long userId, String newPassword);
}

```

- **읽기 전용 기능(UserReadService)과 쓰기 전용 기능(UserWriteService)을 분리**하여 필요 없는 메서드 의존성을 줄임.

```java
@Service
public class UserServiceImpl implements UserReadService, UserWriteService {
    @Override
    public User findUserById(Long userId) {
        return new User(); // 예제이므로 단순한 리턴 값
    }

    @Override
    public List<User> findAllUsers() {
        return new ArrayList<>();
    }

    @Override
    public void registerUser(User user) {
        System.out.println("회원 가입 완료");
    }

    @Override
    public void deleteUser(Long userId) {
        System.out.println("회원 삭제 완료");
    }

    @Override
    public void changeUserPassword(Long userId, String newPassword) {
        System.out.println("비밀번호 변경 완료");
    }
}

```

- `UserServiceImpl`이 **두 개의 인터페이스(UserReadService, UserWriteService)를 구현**하여 필요에 따라 인터페이스를 확장할 수 있음.
- `UserReadService`만 필요한 경우, `UserWriteService`를 구현하지 않아도 됨.

## ⭐ 의존관계 역전 원칙(Dependency Inversion Principle)

"상위 모듈이 하위 모듈에 의존하는 것이 아니라, 추상화(인터페이스)에 의존해야 한다.”

- 프로그래머는 추상화에 의존해야지 구체화에 의존하면 안된다.
- ‘역할’ 에 의존해야 한다.
    
    ![스크린샷(236).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/c9f74c3f-3df1-4afd-a1e3-445f52ed1bed/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(236).png)
    
    - 운전자가 자동차의 ‘역할’ 대해서만 알면 되지, k3에 대해서 구체적으로 알 필요 없음
- Spring의 **의존성 주입(DI)** 자체가 DIP를 따른다.
- 인터페이스 기반 개발을 통해 의존성을 낮춤.

```java
public interface MessageSender {
    void send(String message);
}

@Component
public class EmailSender implements MessageSender {
    public void send(String message) {
        System.out.println("이메일 발송: " + message);
    }
}

@Service
public class NotificationService {
    private final MessageSender messageSender;

    @Autowired
    public NotificationService(MessageSender messageSender) {
        this.messageSender = messageSender;
    }

    public void notifyUser(String message) {
        messageSender.send(message);
    }
}

```

- `NotificationService`는 `MessageSender` 인터페이스에 의존하기 때문에 `EmailSender` 대신 `SmsSender` 등으로 쉽게 교체 가능.