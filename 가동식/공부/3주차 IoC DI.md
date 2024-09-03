# 3주차 IoC, DI

## IoC (Inversion of Control)

> "Don't call us. We'll call you - Hollywood Principle"

### 정의

- 제어의 역전, 프로그램의 제어 흐름 구조가 뒤바뀌는 것
- 객체의 생성이나 제어의 흐름을 직접 관리하는 대신, 외부에서 관리하도록 맡기는 **디자인 원칙**
- 모든 제어권을 프레임워크나 다른 구성 요소에게 위임하게 됨
- Spring Framework에서는 Spring Container가 객체의 생성, 생명 주기, 의존성 주입 등을 관리함
- 프레임워크 or 컨테이너처럼 애플리케이션 컴포넌트의 생성, 관계설정, 사용, 생명주기 관리 등을 관장하는 존재가 필요함
- 해당 개념은 매우 느슨하게 정의되어 폭넓게 사용됨

### 일반적인 프로그램의 흐름

1. 프로그램의 시작지점에서 다음에 사용할 오브젝트를 결정
2. 결정한 오브젝트를 생성
3. 만들어진 오브젝트에 있는 메소드를 호출
4. 오브젝트 메소드 안에서 다음에 사용할 것을 결정
5. 반복...

- 각 오브젝트는 프로그램 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여
- 사용할 구현 클래스를 오브젝트가 결정하고, 그 오브젝트를 필요한 시점에서 생성해서 사용
- 언제 어떻게 사용할 구현 클래스를 만들지를 사용하는 쪽에서 제어하는 구조

### 제어의 역전에서의 흐름

1. 컨테이너 초기화 - 모든 의존성을 관리하고, 오브젝트의 생명주기를 담당
2. 오브젝트마다 의존성 설정 - 구성 파일이나 어노테이션(`@Autowired`)을 사용
3. 오브젝트 생성 및 의존성 주입 - IoC 컨테이너에서 필요한 오브젝트를 생성하고, 의존성을 자동 주입
4. 오브젝트의 사용

- 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않음
- 모든 제어 권한을 본인이 아닌 다른 대상(IoC 컨테이너)에게 위임

### IoC가 적용된 예 - 프레임워크

- 애플리케이션 코드가 프레임워크에 의해 사용
- 프레임워크가 흐름을 주도하는 중 개발자가 애플리케이션 코드를 사용하도록 만드는 방식
- 애플리케이션 코드는 프레임워크가 짜놓은 틀에서 수동적으로 동작해야 함

### 스프링 IoC 용어

#### 빈

- 스프링이 IoC 방식으로 관리하는 오브젝트
- 스프링이 직접 생성과 제어를 담당하는 오브젝트만을 빈이라고 부름

#### 빈 팩토리

- 스프링의 IoC를 담당하는 핵심 컨테이너를 가리킴
- 빈의 등록, 생성, 조회, 반환을 포함한 부가적인 빈을 관리하는 기능을 가짐
- 일반적으로 이를 확장한 애플리케이션 컨텍스트를 이용함
- 빈의 생성과 제어의 관점에서 볼 때 이용

#### 애플리케이션 컨텍스트

- 빈 팩토리를 확장한 IoC 컨테이너
- 빈 팩토리와 기본적인 기능은 동일함
- 스프링이 제공하는 각종 부가 서비스를 추가로 제공함
- `ApplicationContext`는 애플리케이션 컨텍스트가 구현해야하는 기본 인터페이스
  - `BeanFactory`를 상속함

#### 설정정보/설정 메타정보

- 애플리케이션 컨텍스트나 빈 팩토리가 IoC를 적용하기 위해 사용하는 메타정보
- `configuration`이라고 함
- IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성할 때 사용

#### 컨테이너 or IoC 컨테이너

- IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트나 빈 팩토리를 이렇게 부름

#### 스프링 프레임워크

- IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 때 주로 사용함.



### 장점

- 객체 간 결합도를 낮춤
- 유연한 코드 작성이 가능함
- 가독성이 향상됨
- 코드 중복이 감소함
- 유지 보수가 쉬움



## DI (Dependency Injection)

> "의존대상 B가 변하면, 그것이 A에 영향을 미친다." - 토비의 스프링

### 정의

- 의존성 주입
- IoC의 구체적인 구현 방법 중 하나
- 객체가 다른 객체에 대한 의존성을 코드 내부에서 직접 생성하지 않고 외부로부터 주입받는 방법
- 스프링이 제공하는 IoC 방식의 핵심을 짚어주는 용어
- IoC 컨테이너를 DI 컨테이너라고 많이 불림

#### 의존한다

- A가 B에 의존하고 있을 때 B가 변하면 A에 영향을 미친다.
- A에서 B에 정의된 메소드를 호출해서 사용하는 경우
- 방향성이 존재함 - B는 A에 의존하지 않음

#### 조건

- 클래스 모델이나 코드에는 **런타임 시점의 의존관계가 드러나지 않음**
  - **인터페이스**에만 의존하고 있어야 함
- **런타임 시점의 의존관계**는 컨테이너나 팩토리 같은 제 3의 존재가 결정함
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 **외부에서 제공**해줌으로써 만들어줌

### 종류

1. 생성자 주입 - 의존성이 생성자를 통해 주입됨 (권장!!)

``` java
@Controller // 의존성이 주입되는 클래스도 빈으로 등록되어있어야함
public class PostController {
  private final PostService postService;
  
  // 생성자 PostController로 의존성 주입
  @Autowired
  public PostController(PostService postService){
    this.postService = postService
  }
  
  ...
}
```

- 객체가 생성될 때 의존성 주입
- 장점
  - `final` 필드를 사용하여 불변성을 유지할 수 있음
  - 무조건 설정해줘야하기 때문에 누락이 발생하지 않음
- 단점
  - 복잡한 객체의 경우 생성자 매개변수가 많아질 수 있음
  - 순환 참조가 발생할 수 있음
- **가장 권장되는 방식**



2. 세터 주입 - 의존성이 세터 메서드를 통해 주입됨

``` java
@Controller
public class PostController {
  private final PostService postService;
  
  // Setter 메서드로 의존성 주입
  @Autowired
  public void setPostService(PostService postService){
    this.postService = postService
  }
  
  ...
}
```

- 객체가 생성된 후, 별도로 호출되는 세터 메서드로 의존성 주입
- 장점
  - 의존성을 필요에 따라 변경하거나, 주입 시점을 조절할 수 있음
- 단점
  - 세터가 호출되지 않으면 의존성이 주입되지 않은 상태로 객체가 존재할 수 있음
  - 선택적으로 넣거나, 뺄 수 있는 것처럼 보일 수도 있음 (의존적으로 보이지 않을수도 있음)
  - `setter` 메서드로 진행하기 때문에 주입된 의존성의 변경이 가능하여 객체의 불변성 보장이 어려움



3. 필드 주입 - 필드에 직접 의존성을 주입하는 방식

``` java
@Controller
public class PostController {
  
  @Autowired
  private PostService postService; // 필드에 바로 주입
  
  ...
}
```

- 객체가 생성된 후, DI 컨테이너가 필드에 직접 접근하여 주입
- 장점
  - 필드 위에 주입 어노테이션만 추가하면 가능
  - 별도의 생성자나 세터 메서드를 사용하지 않아도 됨
- 단점
  - `private` 필드는 테스트 시 의존성을 주입하기 어려움
  - `final` 필드를 사용할 수 없기 때문에 불변성을 유지하기 어려움
  - 클래스의 의존성을 직접적으로 알기 어려움
  - DI 컨테이너 없이 필드 주입이 불가능함

### 어노테이션

#### @Autowired

- [공식 문서](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)

- 생성자 주입, 세터 주입, 필드 주입에서 사용할 수 있음
- 필요한 의존성을 알아서 주입해줌
- `@Component`로 등록된 빈만 자동으로 주입이 가능함
- 생성자 주입 방식에서 생성자가 1개일 경우 어노테이션을 붙이지 않아도 자동 주입의 대상으로 인식함 (Spring 4.3 이상)

- `@Autowired(required = false)`로 설정하면, 필수가 아닌 것으로 표시하여 후보가 되는 빈이 없을 때 무시함



#### @RequiredArgsConstructor

- Lombok에 포함됨

- DI 방식 중 생성자 주입을 임의의 코드없이 자동으로 설정해주는 어노테이션

- 초기화되지않은  `final` 필드나 `@NonNull`이 붙은 필드에 대해 생성자를 생성함

  - `final`을 사용하지 않으면 `NullPointerException`이 발생할 수 있음
  - 한번 의존성을 주입받은 객체는 프로그램이 끝날 때까지 변하지 않기 때문에 불변성을 표시해주는 것이 좋음

- `@Autowired`를 사용하지 않고 자동으로 의존성을 주입함

- `@RequiredArgsConstructor`를 사용하지 않은 예시

  ``` java
  @Controller
  public class PostController {
    private final PostService postService;
    
    // 생성자 PostController로 의존성 주입
    @Autowired
    public PostController(PostService postService){
      this.postService = postService
    }
    
    ...
  }
  ```

- `@RequiredArgsConstructor`를 사용한 예시

  ``` java
  @Controller
  @RequiredArgsConstructor
  public class PostController {
    // 자동으로 해당 필드의 생성자를 생성하여 주입함
    private final PostService postService;
   
    
    ...
  }
  ```

  

