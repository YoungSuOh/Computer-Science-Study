# AOP(Aspect Oriented Programming)

## AOP란?

- 어플리케이션의 **핵심적인 기능에서 부가적인 기능을 분리 → Aspect**라는 모듈로 만들어서 설계하는 개발 방법

<aside>
💡

Aspect란?

어플리케이션의 **핵심 기능은 아니**지만, 어플리케이션을 구성하는 중요한 요소이고, **부가적인 기능을 담당하는 요소**

Aspect = Advice + Pointcut
Advice : 언제 집어넣을 것이냐 (before(앞에만) / after(뒤에만) / around(앞, 뒤 전부)
Pointcut : 삽입되는 대상자

</aside>

![image (7)](https://github.com/user-attachments/assets/8311c61c-1e64-4c29-81f6-d7e44d9a5060)


- 중복되는 코드들은 `공통 코드,`  각각 identity 있는 코드들을 `핵심 코드`라고 한다.
- 공통 코드들을 빼서 따로 함수로 잡아서 원래 있던 함수들에 삽입을 한다.

![image (8)](https://github.com/user-attachments/assets/ebe4d163-eaf7-4a5f-94e7-bcf104f8ae82)


1. Target
    1. 함수들을 가지고 있는 클래스
    2. 부가기능을 부여할 대상
2. Advice
    1. 부가기능을 담은 모듈
    2. 언제 공통 관심 기능을 핵심 로직에 적용할 지를 정의
    3. Before, After Returning, After Throwing, Around 등이 있다
3. Joinpoint
    1. Advice를 적용 가능한 지점을 의미
4. Pointcut
    1.  조인 포인트를 선별하는 기능을 정의한 모듈
5. Proxy
6. Advisor
7. Aspect
8. Weaving
    1. Advice를 핵심로직코드에 적용하는 것을 Weaving
    2. 스프링은 함수를 호출할 때밖에 안됨
