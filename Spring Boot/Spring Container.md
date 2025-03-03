스프링 컨테이너는 객체(Bean)의 생성, 관리 및 생명 주기를 책임지는 핵심 개념이다.

애플리케이션에서 필요한 객체를 직접 생성하지 않고, **스프링 컨테이너기 대신 객체를 생성하고 관리하며, 필요할 때 객체를 주입**해준다.

## 스프링 컨테이너의 역할

1. 객체(Bean) 생성
    - `@Component`, `@Service`, `@Repository`, `@Controller`, `@Bean` 등의 설정을 기반으로 객체를 생성함
2. 객체의 의존성 주입 (DI)
    - `@Autowired`, `@Qualifier`, `@Resource` 등을 활용하여 필요한 객체를 자동으로 주입해 줌.
3. 객체의 생명주기 관리
    - 객체 생성 → 초기화 → 사용 → 소멸 과정을 관리 (`@PostConstruct`, `@PreDestroy` 사용 가능)
4. 객체 조회 및 제공
    - 필요한 객체를 `ApplicationContext.getBean()`을 통해 가져올 수 있음

## ApplicationContext : 스프링 컨테이너

1. 생성자 주입
    1. 생성자를 통해서 의존 관계를 연결시키는 것을 말한다.
    2. 생성자를 통해서 의존 관계를 연결하기 위해서는 XML 설정 파일에서 <bean>요소의 하위요소로 `<constructor-arg>를` 추가해야 한다.
        1. 전달인자가 2개 이상인 경우
            1. 기본데이터 타입일 경우에는 **`value`** 요소를 사용하여 의존관계를 연결시키기 위한 값을 지정
                
                ![image](https://github.com/user-attachments/assets/2b594a77-68b9-458c-8e24-3adbf43b0231)

                
        2. index 속성을 이용하여 지정
            
            ![image (1)](https://github.com/user-attachments/assets/92822714-70e9-425a-93fc-c1cb7c7c742e)

            
        3. type 속성을 이용하여 지정
            
            ![image (2)](https://github.com/user-attachments/assets/98a973d3-5c7c-4af7-8ba1-dfda82af1c26)

            
        4. 객체를 전달할 경우 **`ref` 요소를 사용**
            
            ![image (3)](https://github.com/user-attachments/assets/851c9f05-681a-4b84-a565-f39d47606675)

            
2. Setter 주입
    1. setter 메서드를 이용하여 의존관계를  연결시키는 것을 말한다.
    2. **`<property>요소의 name 속성`**을 이용하여 값의 의존 관계를 연결시킬 대상이 되는 필드값을 지정한다
        1. 전달인자가 2개 이상인 경우
            
            ![image (4)](https://github.com/user-attachments/assets/9e53044c-c005-470e-a4f4-d21efd77ca8d)

            
        2. 객체를 전달할 경우 **`ref` 요소를 사용**
            
            ![image (5)](https://github.com/user-attachments/assets/6e849def-3ec1-4e1a-9ef9-aad8c181108a)

            
            - Foo와 Bar 빈 등록을 진행
            - ref를 통해 Bar 를 받음

## 스프링 컨테이너를 사용하는 이유

1. **객체의 생명주기를 스프링이 관리** → 필요할 때 생성하고, 필요 없을 때 제거
2. **의존성 주입(DI)으로 결합도를 낮출 수 있음** → 유지보수 용이
3. **싱글톤 패턴 자동 적용** → 동일한 객체를 반복 생성하지 않고 공유
4. **AOP (Aspect-Oriented Programming) 활용 가능** → 로깅, 트랜잭션 관리 쉽게 적용 가능

## 스프링 컨테이너가 객체를 저장하는 메모리

스프링 컨테이너는 **객체(Bean)의 정보를 메모리에 저장하여 관리**하는데, 주로 **Heap 메모리**를 사용한다.

### **Bean의 저장 위치**

스프링에서 관리하는 객체(Bean)는 다음과 같은 방식으로 저장됨.

- **Singleton 스코프 (기본값)**
    - 한 개의 Bean 인스턴스만 생성되어 컨테이너 내에서 공유됨.
    - **Heap 메모리의 Static 영역에 저장됨** (JVM 종료 시 제거됨).
- **Prototype 스코프**
    - 요청할 때마다 새로운 Bean 인스턴스를 생성함.
    - **Heap 메모리의 일반 영역(Young/Old Generation)에 저장됨**.
- **Request, Session, Application 스코프 (웹 환경)**
    - 각각 HTTP 요청, 세션, 애플리케이션 범위 내에서 객체를 관리.
