# JPA 트랜잭션

- Repository 인스턴스의 CRUD Method들은 기본적으로 Transactional함

- 읽기 작업에서는 readOnly flag가 true로 설정됨
- 다른 모든 작업은 일반 @Transactional로 설정되어 트랜잭션이 적용됨

- private는 @Transactional이 적용되지 않음



## @Transactional은 Proxy로 동작한다

### Proxy의 역할

<img src="./2주차 AOP, 프록시 관련.assets/image-20240827010729812.png" alt="image-20240827010729812" style="zoom:50%;" />

1. 접근의 제어 (권한 관리, 흐름 관리) - 프록시 패턴
2. 부가 기능 (로그, 트랜잭션) - 데코레이터 패턴

#### Proxy의 장점

1. OCP: 기존 코드를 변경하지 않고 새로운 기능을 추가 가능
2. SRP: 기존 코드가 해야 하는 일만 유지 가능
3. 기능 추가, 접근 제어도 가능함

#### 하지만 Proxy를 직접 구현하기에는 불편하다

1. 매번 새로운 클래스를 정의해야함
2. 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거로움
3. 부가기능 코드의 중복이 발생함 -> 모든 메서드에 적용해줘야하기 때문에

### Spring에 존재하는 프록시 구현체

ProxyFactoryBean에서 인터페이스의 유무를 확인하여 어떤 프록시를 실행할지 선택하게 됨

- 인터페이스가 있으면 JDK PROXY, 없으면 CGLib

  > ProxyFactoryBean: 프록시를 Bean으로 만들어주는 기능
  >
  > 타깃의 인터페이스 정보가 필요없음, 프록시 빈을 생성해줌

#### 1. JDK PROXY(Dynamic Proxy)

- Spring AOP의 근간

- 만들기 번거로운 프록시 오브젝트를 쉽게 구현하도록 하는 기능
- Proxy Factory에 의해 Runtime에서 다이내믹하게 만들어지는 오브겍트
  - 프록시 팩토리에 인터페이스 정보만 제공하면 해당 인터페이스를 구현한 클래스 오브젝트를 자동으로 생성함
- 자바의 reflection의 Proxy 클래스가 동적으로 프록시를 생성함
  - reflection - 구체적인 클래스 타입을 몰라도 런타임에 클래스에 접근할 수 있는 API
    - JVM의 Optimizer가 동작하지 않아 성능상 느려짐
- 타겟의 인터페이스를 기준으로 Proxy를 생성
  - **인터페이스가 반드시 존재해야함**
- InvocationHandler를 재정의한 Invoke를 구현해야 부가기능이 동작
  - InvocationHandler는 타겟에 의존적이기 때문에 타겟의 수만큼 Bean이 생성됨

##### 생성방식

1. 타겟의 인터페이스를 검증해 ProxyFactory에 의해 타겟의 인터페이스를 상속한 Proxy 객체를 생성
2. Proxy 객체에 InvocationHandler를 포함해서 하나의 객체로 반환
   - InvocationHandler - 부가기능을 작성하는 부분, 중복되는 코드를 줄일 수 있음

<img src="./2주차 AOP, 프록시 관련.assets/image-20240827022002655.png" alt="image-20240827022002655" style="zoom:50%;" />

#### 2. CGLib Proxy

- 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어짐
- **클래스 상속을 이용하여 생성**하기 때문에 인터페이스가 존재하지 않아도 구현 가능
- MethodInterceptor를 재정의한 Intercept를 구현해야 부가기능이 동작

- net.sf.cglib.proxy.Enhancer 의존성을 추가해야함 - spring 3.2의 core에 포함됨
- Default 생성자가 필요 - 4.0의 Objensis 라이브러리에 포함됨
- 타겟의 생성자가 두 번 호출됨 - 4.0의 Objensis 라이브러리에 포함됨





### @Transactional

- 애플리케이션에서는 커넥션 풀을 통해 커넥션 객체를 가져옴 -> 해당 객체로 요청을 보냄
- 각 객체별로 하나의 세션으로 인식하기 때문에 별개의 요청으로 생각함

#### 동작

``` java
public ResponseDto create(RequestDto request) throws SQLException {
  Connection connection = dataSource.getConnection(); // Connection 정보를 가져옴
  connection.setAutoCommit(false); // Transaction 시작
  
  try {
    
    // Business Logic
    Tmp tmp = tmpRepository.save(connection, request);
    TmpBox tmpBox = tmpBoxRepository.save(connection, tmp.getId());
    ResponseDto response = new ResponseDto(tmp.getId());
    
    connection.commit(); // 정상 실행시 Commit
  } catch (Exception e) {
    connection.rollback();  // 예외 상황시 RollBack
    throw new TmpException();
  }
}
```

##### 문제점

1. 중복 코드가 많아짐 - Transaction이 필요한 곳마다 해당 로직을 다 작성해야함
2. 특정 기술에 종속적인 코드가 되어버림
3. 주된 관심사가 아닌 코드가 서비스 레이어에 담김

-> Proxy 객체로 구현하여 사용



- EntityManager를 Injection 받아서 실제 개발자가 작성한 클래스의 메소드 전, 후에 Transaction에 해당하는 begin, commit을 사용하여 동작하도록 구현됨
- JPA의  Transactional은 AOP를 사용하여 Proxy 객체의 형태로 동작함
- 따라서 외부에서 접근이 불가능한 private 메소드는 @Transactional 설정이 불가능한 것



<img src="./2주차 JPA Transaction.assets/image-20240827142646955.png" alt="image-20240827142646955" style="zoom:50%;" />