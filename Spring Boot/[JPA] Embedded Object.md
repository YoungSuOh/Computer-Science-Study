# [JPA] Embedded Object

JPA에서는 Embedded Object란 엔티티의 일부로 사용되는 객체를 의미하며, 해당 객체는 데이터베이스 테이블에서 **독립적인 테이블로 생성되지 않고 부모 엔티티의 테이블 컬럼으로 포함된다.**

즉, **값 타입(Value Type)으로** 사용되며, 엔티티 간 재사용 가능한 값 객체를 모델링할 때 유용

JPA에서는 `@Embeddable`과 `@Embedded` 애너테이션을 사용하여 Embedded Object를 정의하고 활용할 수 있다.

## Embedded Object 사용법

1. Embedded Object 정의 (`@Embeddable` 사용)
    1. Embedded Object는 독립적인 엔티티가 아닌, 다른 엔티티 내부에서 사용될 **값 타입 객체**
        
        ```java
        import jakarta.persistence.Embeddable;
        
        @Embeddable
        public class Address {
            private String city;
            private String street;
            private String zipcode;
        
            // 기본 생성자 필요
            public Address() {}
        
            public Address(String city, String street, String zipcode) {
                this.city = city;
                this.street = street;
                this.zipcode = zipcode;
            }
        
            // Getter, Setter
            public String getCity() { return city; }
            public String getStreet() { return street; }
            public String getZipcode() { return zipcode; }
        }
        ```
        
        - `@Embeddable` 애너테이션을 사용하여 **값 타입**을 정의한다.
        - 엔티티가 아니므로 `@Id`를 사용하지 않는다.
        - 반드시 **기본 생성자**가 필요한다.
2. 엔티티에서 Embedded Object 사용 (`@Embedded` 또는 `@AttributeOverrides`)
    
    ```java
    import jakarta.persistence.*;
    
    @Entity
    public class Member {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        
        private String name;
    
        @Embedded
        private Address address; // Embedded Object 사용
    
        public Member() {}
    
        public Member(String name, Address address) {
            this.name = name;
            this.address = address;
        }
    
        // Getter, Setter
        public Long getId() { return id; }
        public String getName() { return name; }
        public Address getAddress() { return address; }
    }
    ```
    
    - `@Embedded`를 사용하여 **Address 값 타입을 포함한다.**
    - Embedded Object의 필드들은 `Member` 테이블에 다음과 같이 **컬럼으로 추가된다.**
        
        ```sql
        CREATE TABLE Member (
            id BIGINT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(255),
            city VARCHAR(255),
            street VARCHAR(255),
            zipcode VARCHAR(255)
        );
        ```
        
3. 필드명 중복 해결 (`@AttributeOverrides` 사용)
    - 하나의 엔티티에서 같은 Embedded Object를 두 번 이상 사용해야 한다면, 컬럼명이 충돌할 수 있다.
        - `@AttributeOverrides`와 `@AttributeOverride`를 사용하여 개별 컬럼명을 지정
    
    ```java
    import jakarta.persistence.*;
    
    @Entity
    public class Order {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        @Embedded
        @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "shipping_city")),
            @AttributeOverride(name = "street", column = @Column(name = "shipping_street")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "shipping_zipcode"))
        })
        private Address shippingAddress;
    
        @Embedded
        @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "billing_city")),
            @AttributeOverride(name = "street", column = @Column(name = "billing_street")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "billing_zipcode"))
        })
        private Address billingAddress;
    
        public Order() {}
    
        public Order(Address shippingAddress, Address billingAddress) {
            this.shippingAddress = shippingAddress;
            this.billingAddress = billingAddress;
        }
    }
    
    ```
    
    - `Order` 테이블 생성 예시
        
        ```sql
        CREATE TABLE Order (
            id BIGINT AUTO_INCREMENT PRIMARY KEY,
            shipping_city VARCHAR(255),
            shipping_street VARCHAR(255),
            shipping_zipcode VARCHAR(255),
            billing_city VARCHAR(255),
            billing_street VARCHAR(255),
            billing_zipcode VARCHAR(255)
        );
        ```