# Spring MVC (DispatcherServlet)

- 스프링 MVC도 컨트롤러를 사용하여 클라이언트의 요청을 처리한다.
- 스프링에서 **DispatcherServlet 이 MVC에서 C(Control) 부분을 처리**한다.

## 스프링 MVC의 구성 요소

![image (6)](https://github.com/user-attachments/assets/ebbcf9f5-80de-4f00-b135-bd77dc9d00c0)


1. 클라이언트 요청 (HTTP Request)
2. DispatcherServlet이 요청을 수신
    1. DispatcherServlet : Spring MVC에서 **프론트 컨트롤러(Front Controller)** 역할을 하는 핵심 서블릿으로 모든 HTTP 요청을 받아서 적절한 컨트롤러로 전달하고, 응답을 생성하는 역할을 한다.
    
    ```java
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern> <!-- 모든 요청을 받음 -->
    </servlet-mapping>
    ```
    
3. 핸들러 매핑(HandlerMapping)에서 적절한 컨트롤러 찾기
4. 컨트롤러(Controller) 실행
    - `HandlerAdapter`가 해당 컨트롤러의 메서드를 실행하고, 결과를 반환함.
        
        ```java
        @GetMapping("/hello")
        public String hello() {
            return "helloView";
        }
        ```
        
        - 컨트롤러는 보통 `String` 형태의 **뷰 이름(View Name)** 을 반환한다.
5. View Resolver를 통해 View 찾기
    - 컨트롤러가 반환한 뷰 이름을 기반으로, 적절한 `View Resolver`가 뷰를 찾아준다.
        
        ```java
        return "helloView"; // View Resolver가 해당 뷰를 찾음
        ```
        
    - 보통 `.jsp`, `.html` 같은 파일을 뷰로 찾게 됨.
        
        ```java
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/views/" />
            <property name="suffix" value=".jsp" />
        </bean>
        ```
        
6. View 렌더링 후 응답 반환
    - 최종적으로 View가 렌더링되고, HTTP 응답이 클라이언트에게 반환됨.
