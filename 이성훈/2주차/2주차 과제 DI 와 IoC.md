# DI 와 IoC

### DI (Dependency Injection) : 의존성 주입

다음과 같은 경우, `A가 B에 의존한다.` 라고 할 수 있다. B가 변경되면 A의 내용도 변경되기 때문이다.

또한, 의존성을 외부에서 주입하기 때문에 DI(의존성 주입)이 적용된다고 할 수 있다.

```jsx
public class A {
    private B b;
    
    public A(B b) {
        this.b = b;
    }
}
```

- DI가 없는 경우, 클래스 내부에서 의존할 객체를 직접 생성한다. 즉, 변경이 발생할 경우 유연하지 않다는 문제가 있다.
- 반면, DI 가 적용된 경우, 즉 외부에서 인자를 받아 객체를 생성하고 초기화하기 때문에 변경에 유연한 코드라고 할 수 있다.

```jsx
// DI가 적용되지 않은 경우
public class BookService {
	private BookRepository bookRepository = new CustomBookRepository();
	private UserRepository userRepository = new CustomUserRepository();
}

// DI가 적용된 경우
public class BookService {
	private BookRepository bookRepository;
	private UserRepository userRepository;
	
	// BookRepository 인터페이스를 구현한 어떤 클래스든 외부에서 주입해줄 수 있다.
	// UserRepository 인터페이스를 구현한 어떤 클래스든 외부에서 주입해줄 수 있다.
	public BookService(BookRepository bookRepository, UserRepository userRepository) {
		this.bookRepository = bookRepository;
		this.userRepository = userRepository;
	}
}
```

<aside>
💡

**토비의 스프링에 따르면, 의존성 주입을 다음과 같이 하고 있다.**

- 클래스 모델이나 코드에는 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스만 의존하고 있어야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리같은 제 3의 존재가 결정한다.
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.
</aside>

의존성 주입은 세 가지 방법이 있다. **생성자 주입, Setter 주입, 인터페이스 주입**이다.

여기서 스프링에서 권장하는 주입 방법은 **생성자 주입 방법**이다.

**생성자 주입 방법**

```jsx
public class A {
    private B b;
    
    public A(B b) {
        this.b = b;
    }
}
```

**Setter 주입 방법**

```jsx
public class A {
    private B b;
    
    public void setB(B b) {
        this.b = b;
    }
}
```

**인터페이스 주입 방법**

```jsx
public interface BInjection {
    void inject(B b);
}

public A implements BInjection {
    private B b;
    
    @Override
    public void inject(B b) {
        this.b = b;
    }
}
```

**DI를 통해 우리가 얻을 수 있는 것은 무엇일까?**

- 코드가 더 깔끔해진다.
- 객체는 의존성을 조회하지 않으며 의존성의 위치나 클래스를 알지 못한다.
- 특히 의존성이 인터페이스 또는 추상 기본 클래스에 있는 경우, 클래스를 더욱 테스트하기가 쉬워지며, 단위테스트에서 스텁 또는 모의 구현을 사용할 수 있다.

### IoC (Inversion of Control) : 제어의 역전

제어 : 어떠한 클래스 내부에서 다른 객체를 이용할 때, 직접 코드를 생성하여 ‘제어’한다고 한다.

역전 : 객체를 클래스 내부에서 직접 생성하고 제어하는 것이 아니라, 외부에서 인자로 받아 초기화하는 것

- 제어의 역전이 적용되지 않는, 우리가 평소에 프로그래밍 할 경우를 떠올려보자.
    
    우리는 객체의 생명주기(생성 및 초기화, 소멸, 메서드 호출 등)을 구현 객체가 직접 관리한다. 또한 외부 코드(라이브러리)를 호출하더라도 해당 코드의 호출 시점 역시 직접 관리한다.
    
- 반면, 스프링과 같은 프레임워크에서 우리는 Controller, Service 같은 객체들의 동작을 우리가 직접 구현하기는 하지만, 해당 객체들이 어느 시점에 호출될 지는 신경쓰지 않는다. 단지 프레임워크가 요구하는대로 객체를 생성하면 프레임워크가 해당 객체들을 가져다가 생성하고, 메서드를 호출하고 소멸시킨다. 프로그램의 제어권이 역전된 것이다.

프레임워크와 라이브러리에는 어떤 차이가 있는지에 대해 IoC로 설명이 가능하다. 라이브러리를 사용하는 경우에는 어플리케이션의 제어 흐름을 라이브러리에 내주지 않는다. 하지만 프레임워크를 사용한 어플리케이션의 경우, 어플리케이션 코드에 작성한 객체들을 프레임워크가 필요한 시점에 가져다가 사용하기 때문에 제어권의 차이로 구분할 수 있다.

JUnit 과 같은 테스트 프레임워크의 경우에도 `@BeforeEach` 혹은 `@AfterEach`, `@Test`를 통해 테스트에 대한 제어권을 JUnit이 가진다. 

IoC를 통해 우리가 얻을 수 있는 것은 무엇일까?

- 프로그램의 진행 흐름과 구체적인 구현을 분리시킬 수 있다.
- 개발자는 비즈니스 로직에 집중할 수 있다.
- 구현체 사이의 변경이 용이하다.
- 객체간 의존성이 낮아진다.