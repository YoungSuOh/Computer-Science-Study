# Maven & Gradle

**Maven**과 **Gradle**은 Java 프로젝트에서 **빌드 및 의존성 관리**를 위해 사용하는 도구이다.

## Maven

- XML(`pom.xml`) 기반으로 선언적이고 직관적이지만, **설정이 길어질 수 있음**
- `mvn package`, `mvn install` 같은 명령어로 **빌드 과정이 단순**
- Java, Spring 프로젝트에서 사용됨

- `pom.xml`
    
    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.0.0</version>
        </dependency>
    </dependencies>
    ```
    

## Gradle 특징

- Groovy/Kotlin DSL 기반 (`build.gradle`)으로 **더 간결한 문법**
- **Spring Boot 2.5+ 프로젝트에서 Gradle을 더 추천**

- `build.gradle`
    
    ```groovy
    dependencies {
        implementation 'org.springframework.boot:spring-boot-starter-web:3.0.0'
    }
    ```