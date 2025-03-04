# DI (Dependency Injection)

## DI 란?

- 객체간의 의존 관계를 개발자가 직접 설정하는 것이 아니라 외부 컨테이너가 자동으로 주입해주는 방식
- 즉, 객체가 직접 필요한 의존성을 생성하는 것이 아니라 외부에서 생성된 객체를 주입받는 방식이다.

**➡️ DI의 핵심 목적**

- 객체 간의 결합도를 낮추고(**Loosely Coupled**),
- 유지보수와 확장성을 높이며(**High Maintainability & Extensibility**),
- 테스트를 용이하게 만든다(**Easier Unit Testing**).

## DI(Dependency Injection)가 왜 필요한가?

개발자가 직접 객체를 생성하고 의존 관계를 설정하면 안 될까? 가능은 하지만, 유지보수성과 확장성이 떨어진다.

### DI가 나오기 전의 문제점

- 기존 방식
    
    ```java
    public class OrderService {
        private PaymentService paymentService = new PaymentService(); // 직접 객체 생성
    
        public void processOrder() {
            paymentService.pay();
        }
    }
    ```
    
    - 객체 간 결합도가 높아짐
        - `OrderService`는 항상 `PaymentService`를 직접 생성해야 한다
        - 만약 `PaymentService`가 `KakaoPayService`로 바뀌면 `OrderService`도 수정해야 한다.
    - 코드 수정이 어렵고 확장성이 떨어짐
        - 새로운 결제 서비스(`TossPayService`)를 추가하려면 기존 코드를 수정해야 한다.
        - **OCP(개방-폐쇄 원칙)** 위반 → 기존 코드를 변경하지 않고 확장 가능해야 함.
    - 테스트가 어려움
        - `PaymentService`를 직접 생성하므로 Mock 객체로 대체하기 어렵다.
        - `new PaymentService()`를 제거할 수 없으므로 단위 테스트(Unit Test)가 어려워짐.

## DI를 통해 기존 문제점 해결 흐름

- 기존에는 다음과 같이 `OrderServiceImpl` 에서 코드를 직접 변경해주었어야 했다.
    
    ```java
    public class OrderServiceImpl implements OrderService{
        // private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
           private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
        }
    ```
    
    - 이렇게 되면 추상 클래스가 아닌 구체 클래스에 의존하므로 DIP 위반이다. 또한 변경에는 닫혀 있지 않기 때문에 OCP도 위반했다고 볼 수 있다.
    - 현재 의존 관계
        
        ![스크린샷(245).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/864173fa-c521-4af8-8790-194140b140f9/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(245).png)
        

- 해결 방법 → AppConfig (외부 설정 파일)
    
    ```java
    public class AppConfig {
        @Bean
        public MemberService memberService(){
            System.out.println("call AppConfig.memberService");
            return new MemberServiceImpl(memberRepository());
        }
        @Bean
        public MemberRepository memberRepository(){ // DB 설정 분리
            System.out.println("call AppConfig.orderService");
            return new MemoryMemberRepository();
        }
        @Bean
        public OrderService orderService(){
            System.out.println("call AppConfig.memberRepository");
            return new OrderServiceImpl(memberRepository(),discountPolicy());
        }
        @Bean
        public DiscountPolicy discountPolicy(){ // 정책 설정 분리 
            /*return new RateDiscountPolicy();*/
            return new FixDiscountPolicy();
        }
    }
    
    ```
    
    - 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스를 만들자. 위 코드는 생성자 주입 과정이다.
    - 이렇게 되면 Service는 변경할 필요 없이 `discountPolicy()` 부분만 변경해줘서 정책을 바꿔주면 된다.
    - 전체적인 구조
        
        ![스크린샷(249).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/bf4263cf-1385-4d10-a15f-22bf693038c5/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7(249).png)
        

## 의존성 주입 방법

### 생성자 주입 (Constructor Injection)

생성자를 통해 객체의 의존성을 주입하는 방식

- 특징
    - **불변성(Immutable) 보장:** 필드가 `final`로 선언되면 변경할 수 없다.
    - **순환 참조(Circular Dependency) 방지:** 스프링이 순환 참조를 감지하여 애플리케이션 실행 전에 오류를 발생시킨다.
    - **테스트 용이성:** 생성자를 통한 주입이므로 **Mockito 등의 프레임워크를 사용해 쉽게 Mock 객체를 주입할 수 있다.**

✅ **사용 예시**

```java
@Component
public class OrderService {
    private final PaymentService paymentService;

    @Autowired  // 스프링 4.3 이상부터는 생략 가능
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### 필드 주입 (Field Injection)

클래스의 필드에 `@Autowired`를 사용하여 직접 의존성을 주입하는 방식

- 특징
    - 코드가 간결하지만 **테스트가 어렵고**
    - 의존성이 숨겨져 **명확한 객체 생성 흐름을 파악하기 어렵다.**
    - **Spring Container 없이 객체를 생성할 수 없다.** (즉, 순수한 Java 코드로는 테스트가 어려움)

✅ **사용 예시**

```java
@Component
public class OrderService {
    @Autowired
    private PaymentService paymentService; // 아직 어떤 paymentService를 사용할지 결정 안남
}
```

**⚠️ 필드 주입의 단점**

1. **의존성 변경이 어려움** → `private` 필드라서 **Setter를 추가하지 않는 한 변경 불가능**
2. **테스트가 어려움** → `new OrderService()`를 호출할 경우 `paymentService`는 `null`
3. **순환 참조 문제가 발생할 가능성** → 생성자 주입은 앱 실행 시 에러를 발생시키지만, 필드 주입은 런타임에 `NullPointerException` 발생

### 세터 주입 (Setter Injection)

`@Autowired`를 붙인 `setter` 메서드를 통해 의존성을 주입하는 방식

- 특징
    - 객체 생성 후에도 **의존성 변경이 가능**
    - 선택적 의존성을 주입할 때 유용
    - **Setter를 사용해야 하므로 가변성이 증가하고**, 순수 Java 코드에서 명시적으로 `setXXX()`을 호출해야 한다

✅ **사용 예시**

```java
@Component
public class OrderService {
    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**⚠️ 세터 주입의 단점**

1. **객체가 완전히 생성되기 전에 의존성이 변경될 가능성 있음** → Setter가 호출되지 않으면 `null`이 될 수도 있음
2. **객체의 상태가 변경될 가능성 증가** → 불필요한 Setter 메서드가 많아질 수 있음