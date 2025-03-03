# Spring Bean

**Spring Bean**은 **Spring 컨테이너가 관리하는 객체**를 의미한다.

Spring 애플리케이션에서는 일반적으로 객체를 직접 생성하지 않고, Spring 컨테이너가 객체를 생성하고 관리하도록 한다.

## Spring Bean의 특징

1. **Spring 컨테이너가 생성 및 관리**
    - 개발자가 `new` 키워드로 객체를 생성하지 않고, Spring이 객체를 관리함.
2. **싱글톤(Singleton) 기본 적용**
    - 기본적으로 Bean은 **싱글톤**으로 생성됨. 즉, 하나의 인스턴스만 생성되어 애플리케이션에서 공유됨.
3. **의존성 주입(Dependency Injection, DI) 지원**
    - `@Autowired` 등을 활용해 다른 Bean을 주입할 수 있음.
4. **라이프사이클 관리 가능**
    - `@PostConstruct`, `@PreDestroy` 등을 사용해 Bean의 생성과 종료 시점을 제어 가능.

## Spring Bean 등록 방법

1. `@Component` 기반 등록
    - `@Component` 또는 이를 확장한 `@Service`, `@Repository`, `@Controller` 등을 사용하면 자동으로 Bean이 등록된다.
2. `@Bean` 기반 등록 (Java Config)
    - `@Configuration` 클래스에서 `@Bean`을 사용해 수동으로 등록 가능
        
        ```java
        @Configuration
        public class AppConfig {
            @Bean
            public MyComponent myComponent() {
                return new MyComponent();
            }
        }
        
        ```
        
3. XML 기반 등록 (예전 방식)
    - applicationContext.xml 파일 생성
        - Namespaces
            
            ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7c90871f-462d-4f71-a192-56327a9e079c/8f9e4d2b-5b9b-412f-9b32-3a20a41dda3f/image.png)
            
    - bean 정보 입력
        - bean 정보
            - MessageBeanEn
                
                ```java
                public class MessageBeanEn implements MessageBean {
                	public MessageBeanEn() {
                		System.out.println("MessageBeanEn의 기본 생성자");
                	}
                	@Override
                	public void sayHello(String name) {
                		System.out.println("Hello "+name);
                		
                	}
                
                }
                
                ```
                
            - MessageBeanKo
                
                ```java
                public class MessageBeanKo implements MessageBean{
                	public MessageBeanKo() {
                		System.out.println("MessageBeanKo의 기본 생성자");
                	}
                	@Override
                	public void sayHello(String name) {
                		System.out.println("안녕 "+name);		
                	}
                
                }
                
                ```
                
        - 빈 등록
            
            ```java
            <?xml version="1.0" encoding="UTF-8"?>
            <beans xmlns="http://www.springframework.org/schema/beans"
            	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            	xmlns:context="http://www.springframework.org/schema/context"
            	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
            	<bean id="messageBeanEn" class="sample01.MessageBeanEn"></bean>
            	<bean id="messageBeanKo" class="sample01.MessageBeanKo"></bean>
            
            </beans>
            
            ```
            
            - MessageBeanEn, MessageBeanKo
        - 빈 호출
            
            ```java
            public class HelloSpring {
            	public static void main(String args[]) {
            		// 1대 1 반응 -> 결합도 100%
            		MessageBeanEn messageBeanEn = new MessageBeanEn();
            		messageBeanEn.sayHello("spring");
            		System.out.println("=========================================");
            		
            		// 부모 = 자식 -> 다형식, 결합도 낮추기
            		MessageBean messageBean = new MessageBeanKo();
            		messageBean.sayHello("spring");
            		System.out.println("=========================================");
            		
            		
            		// 스프링 설정파일
            		ApplicationContext context = new FileSystemXmlApplicationContext("src/applicationContext.xml"); // 모든 빈 생성 -> 기본 생성자 호출됨
            		**ApplicationContext context = new  ClassPathXmlApplicationContext("applicationContext.xml");**
            		MessageBean messageBean2 = (MessageBean)context.getBean("messageBeanEn"); // applicationContext.xml에서 Bean의 id값을 넣어준다.Object type
            		messageBean2.sayHello("spring1");
            		System.out.println("=========================================");
            		
            		// 스프링 설정파일
            		MessageBean messageBean3 = (MessageBean)context.getBean("messageBeanKo"); // applicationContext.xml에서 Bean의 id값을 넣어준다.Object type
            		messageBean3.sayHello("spring2");
            		System.out.println("=========================================");
            	}
            }
            ```
            

## Spring Bean 사용 방법

1. `@Autowired`를 사용한 의존성 주입
    
    ```java
    @Component
    public class MyService {
        
        private final MyComponent myComponent;
    
        @Autowired
        public MyService(MyComponent myComponent) {
            this.myComponent = myComponent;
        }
    
        public void execute() {
            myComponent.doSomething();
        }
    }
    ```
    
2. ApplicationContext에서 직접 가져오기
    - Spring의 `ApplicationContext`를 사용하여 Bean을 직접 가져올 수도 있음
    
    ```java
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MyComponent myComponent = context.getBean(MyComponent.class);
        myComponent.doSomething();
    }
    ```
    
    - 하지만, 대부분의 경우 `@Autowired`를 이용한 DI 방식을 추천함.

## Spring Bean의 라이프사이클

Spring Bean의 생명주기는 크게 다음과 같은 단계로 이루어짐.

1. **객체 생성**
2. **의존성 주입 (@Autowired)**
3. **초기화 메서드 실행 (`@PostConstruct`)**
4. **사용됨**
5. **소멸 전 메서드 실행 (`@PreDestroy`)**
6. **Bean 소멸**

```java
@Component
public class MyComponent {

    @PostConstruct
    public void init() {
        System.out.println("Bean이 생성되었습니다!");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Bean이 소멸됩니다!");
    }
}
```

> @PostConstruct는 Bean이 생성될 때, @PreDestroy는 Bean이 소멸되기 전에 실행됨.
> 

## Bean Scope

Spring에서는 Bean의 생명주기를 제어하기 위해 **Scope(범위)**를 설정할 수 있다.

기본적으로 **싱글톤(Singleton)**이지만, 필요에 따라 여러 가지 스코프를 적용할 수 있다.

- 싱글톤  : @Scope("singleton")
    - 스프링의 기본 스코프. 빈이 컨테이너 당 하나만 생성되어 모든 요청에서 동일한 인스턴스를 사용.
    - **⭐ 스프링이 실행 될 경우 바로  스프링 컨테이너에 빈이 등록이 된다.**
- 프로토타입 : @Scope("prototype")
    - 빈을 요청할 때마다 새로운 객체가 만들어지므로 상태를 가질 수 있지만, 관리가 더 복잡할 수 있음.
    - ⭐ **스프링이 실행 될 경우 바로  스프링 컨테이너에 빈이 등록되는 것이 아닌, 인스턴스 호출이 될 경우 new 처럼 생성이 됐다가 소멸을 한다.**
- 리퀘스트 : @Scope(value = WebApplicationContext.SCOPE_REQUEST)
    - 웹 애플리케이션에서 주로 사용되며, 각 HTTP 요청마다 독립적인 인스턴스가 제공됨
- 세션 : @Scope(value = WebApplicationContext.SCOPE_SESSION)
    - 사용자 세션 동안 동일한 빈 인스턴스를 사용. 세션이 종료되면 빈도 파괴됨.
- 애플리케이션 : @Scope(value = WebApplicationContext.SCOPE_APPLICATION)
    - 애플리케이션 전역에서 공유되는 빈으로, 서블릿 컨텍스트가 살아있는 동안 유지됨.
- 웹소켓 : @Scope(value = "websocket")
    - WebSocket 세션 당 빈이 생성되고, 세션이 종료되면 빈이 파괴됨.

### 등록 방법

1. `@Scope` 어노테이션 사용
    
    ```java
    @Component
    @Scope("singleton")  // 기본값이므로 생략 가능
    // 나머지 prototype, request, session, application
    public class SingletonBean {
        public SingletonBean() {
            System.out.println("Singleton Bean 생성됨!");
        }
    }
    ```
    
2. `applicationContext.xml` XML 설정을 통한 Scope 등록 (옛 방식)
    
    ```java
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
    	<bean id="messageBeanEn" class="sample01.MessageBeanEn"></bean>
    	<bean id="messageBeanKo" class="sample01.MessageBeanKo" scope="prototype"></bean>
    </beans>
    ```
    
3. `@Component` 로 등록
    - `applicationContext.xml`
    
    ```java
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
    	<context:component-scan base-package="sample01"/>
    </beans>
    ```