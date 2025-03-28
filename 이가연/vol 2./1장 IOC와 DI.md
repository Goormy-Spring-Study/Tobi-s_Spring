# 1.1 IOC 컨테이너 : 빈 팩토리와 애플리케이션 컨텍스트

IOC란 제어의 역전을 뜻한다. 즉, 오브젝트 생성과 관계 설정, 사용, 제거 등의 작업을 개발자가 아닌 컨테이너가 담당함으로써 제어가 역전됐다는 뜻이다. 그리고 이 컨테이너를 IOC 컨테이너라고 부른다. 

***IOC 컨테이너 = 빈 팩토리 = 애플리케이션 컨텍스트*** 

- 빈 팩토리는 오브젝트 생성과 오브젝트 사이의 런타임 관계를 설정하는 **DI 관점**에서 바라보는 컨테이너를 뜻한다.
- 애플리케이션 컨텍스트는 빈 팩토리를 상속한다.
- 결국 스프링 컨테이너는 **애플리케이션 컨텍스트 인터페이스를 구현한 클래스의 오브젝트**이다.

```java
public interface ApplicationContext extends ListableBeanFactory, HierarchicalBeanFactory,
MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
};
```

## 1.1.1 IoC 컨테이너를 이용해 애플리케이션 만들기

가장 간단하게 IoC 컨테이너를 만드는 방법은 ApplicationContext 구현 클래스의 인스턴스를 만드느 것이다. 

```java
StaticApplicationContext ac = new StaticApplicationContext();
```

이러면 IOC 컨테이너가 생성됐지만 이 컨테이너는 빈 컨테이너이다. 동작을 위해선 **POJO** 클래스와 **설정 메타 정보**가 필요하다.

### POJO (Plain Old Java Object) 클래스

POJO 클래스는 스프링 프레임워크에 종속되지 않고 다른 POJO와 느슨한 결합을 가진 간단하고 독립적인 Java 객체이다. 

Hello 클래스는 Printer라는 인터페이스에만 의존한다. 런타임시 어떤 구체적인 클래스의 오브젝트를 사용하게 될 지는 상관 없다.

독립적인 POJO 클래스를 만들고 결합도가 낮은 유연한 관계를 가지도록 인터페이스로 연결해주는 것이 IOC 컨테이너가 사용할 POJO를 준비하는 단계이다. 

```java
public class Hello {

    String name;
    Printer printer; 
    
    public String sayHello() {
    	return "Hello "+ name; 		// 프로퍼티로 DI 받은 이름을 이용해 간단한 인사문구 만들기 
    }
	
    public void print() {
      // DI에 의해 의존 오브젝트로 제공받은 Printer타입의 오브젝트에게 출력 작업을 위임
    	this.printer.print(sayHello());		
    }
    
    public void setName(String name) {
    	this.name = name;
    }
    
    public void setPrinter(Printer printer) {
    	this.printer = printer;
    }
}
```

```java
public interface Printer {
    void print(String message);
}

public class StringPrinter implements Printer {
    private StringBuffer buffer = new StringBuffer();
    
    public void print(String message) {
    	this.buffer.append(message);	// 내장 버퍼에 메시지 추가
    }
    
    public String toString() {
    	return this.buffer.toString();	// 내장 버퍼에 추가해둔 메시지를 스트링으로 가져온다. 
    }
}

public class ConsolesPrinter implements Printer{
    public void printer(String message) {
    	System.out.println(message);
    }
}
```

### 설정 메타 정보

그 다음으론 준비된 POJO 클래스 중에서 애플리케이션이 사용할 것을 선정하고 IOC 컨테이너가 제어할 수 있도록 적절한 메타정보를 만들어야 한다. 

IOC 컨테이너의 가장 기초적인 역할은 오브젝트를 생성하고 관리하는 것으로, 이 오브젝트를 **빈**이라고 부른다. 설정 메타 정보는 **이 빈을 어떻게 만들고 어떻게 동작하게 할 것인가**에 관한 정보이다. 

이 메타정보는 BeanDefinition 인터페이스에 담기고 컨테이너는 이를 이용해 IOC 와 DI 작업을 수행한다.

메타 정보 종류는 대략 다음과 같다. 

- **빈 아이디, 이름, 별칭**: 빈 오브젝트를 구분할 수 있는 식별자
- **클래스** 또는 클래스 이름: 빈으로 만들 POJO 클래스 또는 서비스 클래스 정보
- **스코프**: 싱글톤, 프로토타입과 같은 빈의 생성 방식과 존재 범위
- **프로퍼티 값 또는 참조**: DI에 사용할 프로퍼티 이름과 값 또는 참조하는 빈의 이름
- **생성자 파라미터 값 또는 참조**: DI에 사용할 생성자 파라미터 이름과 값 또는 참조할 빈의 이름
- 지연된 로딩 여부, 우선 빈 여부, 자동와이어링 여부, 부모 빈 정보, 빈팩토리 이름 등

![image](https://github.com/user-attachments/assets/77a00d3e-9547-48be-8d78-4fb858ceeff2)

요약 하자면, **스프링 애플리케이션은 POJO 클래스와 설정 메타 정보를 이용해서 IOC 컨테이너가 만들어주는 오브젝트의 조합**인 것이다. 

다음은 두 개의 POJO 클래스를 빈으로 등록하고 DI 한 뒤 두 빈이 서로 연동해서 동작하는지 확인하는 코드이다. 

```java
@Test
public void registerBeanWithDependency(){

    StaticApplicationContext ac = new StaticApplicationContext();

    /* StringPrinter 클래스 타입이며 printer라는 이름을 가진 빈을 등록한다. */
    ac.registerBeanDefinition("printer", new RootBeanDefinition(StringPrinter.class));
    
    BeanDefinition helloDef = new RootBeanDefinition(Hello.class);
    helloDef.getPropertyValue().addPropertyValue("name","Spring");  // 단순 값을 갖는 프로퍼티
    helloDef.getPropertyValue().addPropertyValue("printer", 	// 아이디가 printer인 빈에 대한
        new RuntimeBeanReference("printer"));                   // 레퍼런스를 프로퍼티로 등록
    
    ac.registerBeanDefinition("hello",helloDef);
    
    Hello hello = ac.getBean("hello", Hello.class);
    hello.print();
    
    
    /* Hello 클래스의 print() 메소드는 DI 된 Printer 타입의 오브젝트에게 요청해서 인사말을 출력한다.
       이 결과를 스트링으로 저장해두는 printer 빈을 통해 확인한다. */
    assertThat(ac.getBean("printer").toString(), is("Hello Spring"));

}
```

IOC 컨테이너 종류는 StaticApplicationContext, GenericApplicationContext, GenericXmlApplicationContext, WebApplicationContext 등이 있지만 가장 많이 사용되는 것은 `GenericApplicationContext`와 `WebApplicationContext` 이다. 

그럼 이제 설정 메타 정보를 이용해 빈 오브젝트를 만들었고 DI 시켰으니 애플리케이션을 동작시킬 준비가 끝난 것일까? 

***아니다 !***

이제 준비된 빈 오브젝트의 메서드를 호출함으로써 애플리케이션을 동작시켜야 한다. 이러한 기동 역할을 맡은 빈을 사용하려면 IOC 컨테이너에서 요청해서 빈 오브젝트를 가져와야 하는데, `getBean()` 이라는 메서드로 가져올 수 있다. 이후에는 빈끼리 DI로 연결돼 의존관계를 타고 오브젝트가 호출되면서 애플리케이션이 동작한다. 

## 웹 애플리케이션의 동작 방식은 근본적으로 다르다

독립 자바 프로그램은 JVM에게 main() 메서드를 가진 클래스를 시작시켜달라고 요청해 동작한다. 

하지만 웹에서는 main() 메서드를 호출할 방법이 없다. 

대신 서블릿 컨테이너가 브라우저로 오는 HTTP 요청을 받아서 해당 요청에 매핑되어 있는 서블릿을 실행해주는 방식으로 동작한다. 

즉, 웹에서는 서블릿이 일종의 main() 메서드와 같은 역할을 하는 것이다. 

다음은 웹 애플리케이션에서 스프링 애플리케이션을 가동시키는 방식이다. 

![image](https://github.com/user-attachments/assets/df165524-fb55-4b79-8e98-a04997b8cc07)



1. main() 메서드 역할을 하는 서블릿을 생성한다. 
2. 애플리케이션 컨텍스트를 생성한다. 
3. 요청이 서블릿으로 들어올 때마다 getBean() 메서드로 필요한 빈을 가져와 정해진 메서드를 실행한다.

이 2,3 번의 과정은 (애플리케이션 컨텍스트를 생성하고, 설정 메타정보로 초기화해, 요청마다 적절한 빈을 찾아서 실행해줌) 스프링의 DispatchServlet이 담당한다.

# 1.2 IoC/DI를 위한 빈 설정 메타정보 작성

![image](https://github.com/user-attachments/assets/599f53e0-39c7-4db3-9f1a-2a5f5c4e2258)


XML, 애노테이션, 자바 코드로 작성된 빈 설정 정보가 전용 리더를 통해 읽혀져서 BeanDefinition 타입의 오브젝트로 변환되고 그 정보를 IOC 컨테이너가 활용한다. 

이 중에서 현재 사용되는 방식은 대부분 애노테이션 방식이다. 

```java
@Component
public class AnnotatedHello{
}
```

```java
@Test
    public void 컴포넌트빈주입테스트(){
        ApplicationContext ctx =
                new AnnotationConfigApplicationContext(
                        "study.tobi.ioc.bean");

        AnnotatedHello hello = ctx.getBean("annotatedHello", AnnotatedHello.class);

        Assertions.assertThat(hello).isNotNull();
    }
```

`@Component` 애노테이션이 붙은 클래스는 컨테이너가 관리하는 빈으로 등록된다. 

해당 애노테이션은 **컴포넌트 스캔** 대상이므로, `@ComponentScan`이나 `@SpringBootApplication`에 의한 자동 스캔을 통해서만 등록된다.

```java
@Configuration
public class AnnotatedHelloConfig {

    @Bean
    public AnnotatedHello AnnotatedHello(){
        return new AnnotatedHello();
    }
}
```

```java
@Test
    public void 자바코드빈주입테스트(){

        ApplicationContext ctx = new AnnotationConfigApplicationContext(AnnotatedHelloConfig.class);
        AnnotatedHello hello = ctx.getBean("AnnotatedHello", AnnotatedHello.class);

        AnnotatedHelloConfig config = ctx.getBean("annotatedHelloConfig", AnnotatedHelloConfig.class);
        Assertions.assertThat(config).isNotNull();
    }
```

`@Bean` 도 스프링 빈 정의 애노테이션이지만, 메서드가 반환하는 객체를 빈으로 정의할 때 사용한다. 

`@Configuration`이 붙은 클래스 내에 선언된 메서드 위에 사용된다.

여기서 @Configuration 은 @Bean 애노테이션으로 스프링 빈을 정의하는 설정 클래스임을 나타내는 애노테이션이다. 

한편, @Bean 애노테이션이 정의된 메서드는 `싱글톤`을 보장하기 위해 스프링이 내부적으로 프록시 객체를 사용하기 때문에, **동일한 빈 메서드에 대해 여러번 호출돼도 하나의 인스턴스만이 반환**된다.
