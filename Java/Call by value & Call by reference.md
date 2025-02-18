# Call by value & Call by reference

## call by value

함수가 호출될 때, 메모리 공간 안에서는 함수를 위한 별도의 임시 공간이 생성된다. (종료 시 해당 공간 사라짐)

call by value 호출 방식은 함수 호출 시 전달되는 변수 값을 복사해서 함수 인자로 전달한다.

이때, 복사된 인자는 함수 안에서 지역적으로 사용되기 때문에 local value 속성을 가짐.

👉 따라서 함수 안에서 인자 값이 변경되더라도, 외부 변수 값은 변경 안됨

```cpp
void func(int n) {
    n = 20;
}

void main() {
    int n = 10;
    func(n);
    printf("%d", n);
}
```

> printf로 출력되는 값은 그대로 10이 출력된다
> 

## call by reference

call by reference 호출 방식은 함수 호출 시 인자로 전달되는 변수의 레퍼런스를 전달함

따라서 함수 안에서 인자 값이 변경되면, 인자로 전달된 객체의 값도 변경됨

```cpp
void func(int *n) {
    *n = 20;
}

void main() {
    int n = 10;
    func(&n);
    printf("%d", n);  // 20
}
```

## Java 함수 호출 방식

자바의 경우, 항상 call by value로 값을 넘긴다.

C/C++와 같이 변수의 주소값 자체를 가져올 방법이 없으며, 이를 넘길 수 있는 방법 또한 있지 않다.

reference type(참조 자료형)을 넘길 시에는 해당 객체의 주소 값을 복사하여 이를 가지고 사용한다.

따라서 원본 객체의 Property까지는 접근이 가능하나, 원본 객체 자체를 변경할 수는 없다.

```java
// 1 : a에 User 객체 생성 및 할당(새로 생성된 객체의 주소값을 가지고 있음)
User a = new User("gyoogle");   

// a   -----> User Object [name = "gyoogle"]

// 2 : b라는 파라미터에 a가 가진 주소값을 복사하여 가짐
public void foo(User b){        // 2
	System.out.println(b.getName());  // gyoogle

// a   -----> User Object [name = "gyoogle"]
               ↑     
// b   -----------

// 3 : 새로운 객체를 생성하고 새로 생성된 주소값을 b가 가지며 a는 그대로 원본 객체를 가리킴
    b = new User("jongnan");    // 3
    
// a   -----> User Object [name = "gyoogle"]
                   
// b   -----> User Object [name = "jongnan"]    
}
```

- 예제를 보면 처음에 `User b` 는 a의 주소 값을 복사하여 넘겨주는 방식이라 **call by referencef라고 오해할 수 있지만,** 원본 객체 자체를 변경할 수 없다.

다음은 C/C++와 Java에서 변수를 할당하는 방식을 보면 알 수 있다.

```java

// c/c++ 
 
 int a = 10;
 int b = a;
 
 cout << &a << ", " << &b << endl; // out: 0x7ffeefbff49c, 0x7ffeefbff498
 
 a = 11;
 
 cout << &a << endl; // out: 0x7ffeefbff49c

//java
 
 int a = 10;
 int b = a;
 
 System.out.println(System.identityHashCode(a));    // out: 1627674070
 System.out.println(System.identityHashCode(b));    // out: 1627674070
 
 a = 11;

 System.out.println(System.identityHashCode(a));    // out: 1360875712
```

- C/C++에서는 생성한 변수마다 새로운 메모리 공간을 할당하고 이에 값을 덮어씌우는 형식으로 값을 할당한다. (`*` 포인터를 사용한다면, 같은 주소값을 가리킬 수 있도록 할 수 있다.)
- Java에서 또한 생성한 변수마다 새로운 메모리 공간을 갖는 것은 마찬가지지만, **그 메모리 공간에 값 자체를 저장하는 것이 아니라 값을 다른 메모리 공간(Heap)에 할당하고 이 주소값을 저장**하는 것이다.