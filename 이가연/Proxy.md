![](https://velog.velcdn.com/images/dlrkdus/post/ec62e845-f680-44ea-886c-134a6b96b7af/image.png)
[참고 자료](https://www.youtube.com/watch?v=MFckVKrJLRQ)

지난 번에 AOP에 대해 공부하면서 Spring AOP는 기본적으로 프록시 방식으로 동작한다고 배웠는데, 정확히 어떤 구조로 동작하는건지, Java에서 Proxy는 어떻게 구현하는건지 궁금한 점이 많아져 제대로 조사해봤다. 

우선 공부를 시작하기 전 나는 프록시에 대한 개념을 '대리자' 라는 개념으로써 서버와 클라이언트 사이에서 요청을 가로채 대신 메서드를 호출하고 응답하는 패턴으로 알고 있었다. 
또 스프링의 트랜잭션이 Proxy로 동작한다고도 인지만 하고 있었고, 정확히 어떻게 돌아가는건지는 알지 못했다. 

그럼 AOP에 대한 공부의 연장선으로 Proxy의 동작 구조와 구현 방식을 공부하는 시간을 가져보겠당 !

# Proxy란? 

우선 프록시 패턴에 대해 간단히 얘기하자면 자바 프로그래밍의 디자인 패턴 중 하나로 초기화 지연, 접근 제어, 로깅, 캐싱 등, 기존 대상 원본 객체를 수정 없이 추가 동작 기능들을 가미하고 싶을 때 사용하는 코드 패턴이다. 이 디자인 패턴을 적용하면 개방 폐쇄 원칙의 효과를 얻을 수 있어 코드 수정없이 유연하게 확장이 가능하여 유지보수에 유리하다는 장점이 있다.

![](https://velog.velcdn.com/images/dlrkdus/post/a5671fe2-b116-4f57-9f3a-b44430a98bd6/image.png)

**프록시의 사용 목적**은 크게 2가지로 나뉜다. 

1. 클라이언트가 타겟에 접근하는 방법을 제어하기 위해
2. 타겟에 부가적인 기능을 부여하기 위해

스프링에서 프록시 객체는 클라이언트로부터 타겟을 대신해서 요청을 받는 대리인으로써 실제 오브젝트인 타겟은 프록시를 통해 최종적으로 요청 받아 처리한다. 
이를 통해 타겟은 자신의 기능에만 집중하고 부가기능은 프록시에게 위임한다.



이 프록시 객체는 프록시 패턴을 통해 직접 구현할 수 있다.

## 프록시 구현 방법

![](https://velog.velcdn.com/images/dlrkdus/post/c3d50c02-2b5a-4410-b294-de8bf56cf626/image.png)

**1. 프록시 인터페이스를 구성한다**

```java
public interface ProxyInterface {
	String run();
}
```
**2. 인터페이스를 상속하는 타겟 객체(실제 서비스)와 프록시 객체를 구성한다. 
**

```java
public class Service implements ProxyInterface {
	@Override
	public String run() {
		return "실제 로직입니다.";
	}
}
```
**3. 이때 프록시 객체에는 부가 기능을 부여한다. 
**

```java
public class Proxy implements ProxyInterface {
	ProxyInterface pi;

	@Override
	public String runSomething() {
		System.out.println("부가 기능 부여 및 흐름 제어 목적의 프록시입니다.");
		pi = new Service();
		return pi.run();
	}
}
```

**4. Main class에서 서비스를 직접 호출하지 않고 프록시를 호출한다.
**

```java
public class Main {  	
  public static void main(String[] args) { 		
      //직접 호출하지 않고 프록시를 호출한다. 		
      ProxyInterface proxy = new Proxy(); 		
      System.out.println(proxy.run()); 	
      }
  }
}
```
이를 통해 직접 서비스에 접근하지 않고 프록시를 통해 우회해 접근하는 흐름 제어를 실현하고 있다. 그럼 장점을 하나씩 살펴보자. 

## 프록시의 장점과 단점 

**장점** 

- 개방폐쇄원칙(OCP) : 기존 코드를 변경하지 않고(타겟에 적용하지 않고) 새로운 기능을 추가할 수 있다. 
- 단일책임원칙(SRP) : 기존 코드가 해야 하는 일만 유지할 수 있음
- 기능 추가, 접근 제어 등 

하지만 이런 프록시에도 치명적인 단점이 있다. 

**단점**

- 코드의 복잡도가 증가한다. 
  - 타겟 하나당 인터페이스, 프록시 객체를 생성해줘야 한다. 
- 중복 코드가 발생한다. 
  - 타겟이 여러 개라면? 부가 기능이 별로 없는 프록시라면? 중복 코드가 상당할 것.

이러한 단점을 극복하기 위해 동적 프록시가 등장하였다. 

# 동적 프록시 

대표적인 동적 프록시인 JDK Dynamic Proxy에 대해 소개한다. (기니까 JDK 프록시라 할게요..)
JDK 프록시는 앞서 설명한 것처럼 프록시 객체를 직접 구현하지 않고, 런타임 시점에 프록시 클래스/인터페이스를 생성한다. 이는 java.lang.reflect.Proxy 패키지에서 제공해주는 API로 생성된다. (단, 동적이므로 JVM의 optimization이 작동하지 않아 성능상 느리다)

JDK 동적 프록시의 장점은 다음과 같다. 

- 프록시 클래스를 직접 구현하지 않아도 된다. 
  - 코드 복잡도 해소 
- Invocation Handler 
  - 중복 코드 제거 
  
## Dynamic Proxy 구성 요소 

![](https://velog.velcdn.com/images/dlrkdus/post/9fcbfa70-80e4-44ea-8d1a-382612264426/image.png)



### newProxyInstance()

이 메서드를 호출하면  따로 프록시 클래스 정의 없이 자동으로 프록시 객체를 등록할 수 있다.
프록시 객체 생성을 위해 다음과 같은 3가지의 매개변수를 받는다. 
하면 따로 프록시 클래스 정의 없이 자동으로 프록시 객체를 받는다. 

[코드 출처](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EB%88%84%EA%B5%AC%EB%82%98-%EC%89%BD%EA%B2%8C-%EB%B0%B0%EC%9A%B0%EB%8A%94-Dynamic-Proxy-%EB%8B%A4%EB%A3%A8%EA%B8%B0)
```java
public class Proxy implements java.io.Serializable {

	// ...
    
    public static Object newProxyInstance(
        ClassLoader loader, 
        Class<?>[] interfaces, 
        InvocationHandler h 
    ) throws IllegalArgumentException {
        // ...
    }

}
```
- ClassLoader loader : 프록시 클래스를 만들 클래스 로더, 일반적으로 프록시 객체가 구현할 인터페이스 (앞선 예제로는 ProxyInterface) 에 얻어온다. 
- Class< ? >[] interfaces : 타겟의 인터페이스
- InvocationHandler : 프록시 메소드가 호출되었을 때 실행되는 핸들러 

### Invocation Hanlder 

이 인터페이스의 구성은 invoke() 추상 메소드 하나뿐이다. 

```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

프록시 객체, 메소드 정보, 메소드에 전달된 매개변수 3가지를 매개변수로 받는다. 
이 invoke 메소드는 **동적 프록시의 메소드가 호출되었을 때 이를 낚아채어 대신 실행되는 메서드다.** 프록시의 핵심이라고 할 수 있겠다. 


## JDK Dynamic Proxy의 특징 

- Reflection API 를 사용한다. 
- **인터페이스가 반드시 필요하다. **
- 부가기능 추가를 위해선 Invocation Handler의 incoke() 메소드를 재정의해줘야 한다. 

그런데 이 인터페이스를 구현하지 않고도 프록시를 구현할 수 있는 또다른 방법이 있다. 

# CGLIB 

![](https://velog.velcdn.com/images/dlrkdus/post/6367fa63-1cd6-46c0-83cd-6295173a8b5e/image.png)


스프링에선 클라이언트가 요청을 보냈을 때 인터페이스가 있으면 JDK Dynamic Proxy로, 없으면 CGLIB로 요청을 보낸다. CGLIB는 상속을 통해 프록시를 구현한다. Spring Boot에서는 바로 이 CGLIB로 프록시를 만들고 해당 프록시로 AOP를 구현한다.

> 이유는 스프링부트는 기본적으로 CGLIB를 내장하고 있고 spring.aop.proxy -target-class가 true로 설정되어있기 때문이다.
> false로 바꿔주면 JDK Proxy가 동작한다.

CGLIB의 특징은 다음과 같다. 

- 바이트 코드를 조작해서 프록시를 생성한다.
- 부가 기능을 추가하기 위해선 MethodInterceptor를 재정의한 intercept를 구현해야 한다.

이때 MethodInterceptor은 JDK Proxy의 InvokeHanlder와 달리 타겟을 매개변수로 갖지 않는다. 이는 JDK Proxy의 단점이기도 한데(타겟마다 빈을 생성해줘야 함), CGLIB는 이를 통해 부가 기능을 독립적(싱글톤)으로 유지할 수 있게 한다. 

## CGLIB의 장점과 단점 

**장점**

- 인터페이스 없이 단순 클래스만으로 프록시 객체를 동적으로 생성이 가능하다.
- 리플렉션이 아닌 바이트 조작을 사용하며, 타겟에 대한 정보를 알고 있기 때문에 JDK Dynamic Proxy에 비해 성능이 좋다.

**단점**

- 의존성을 추가해야한다.(Spring 3.2 이후 버전의 경우 Spring Core 패키지에 포함)
- **Default 생성자**가 필요하다. (4.0 objenesis 라이브러리로 해결 가능)
- **타겟의 생성자가 두 번** 호출된다.(4.0 objenesis 라이브러리로 해결 가능) 

CGLIB는 메소드가 처음 호출되었을 땐 동적으로 타겟의 클래스의 바이트 코드를 조작하고 이후 호출엔 조작된 바이트 코드를 재사용한다. 

| JDK 다이나믹 프록시                  | CGLIB                               |
|--------------------------------------|-------------------------------------|
| 01. reflection api를 사용한다.        | 01. 바이트 코드를 조작해서 빠르다.   |
| 02. 인터페이스가 반드시 필요하다.     | 02. 상속을 이용해서 프록시를 만든다. |
|                                      | 03. 메서드에 `final`을 붙이면 안된다. |




#### 출처 
- https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EB%88%84%EA%B5%AC%EB%82%98-%EC%89%BD%EA%B2%8C-%EB%B0%B0%EC%9A%B0%EB%8A%94-Dynamic-Proxy-%EB%8B%A4%EB%A3%A8%EA%B8%B0
- https://www.youtube.com/watch?v=MFckVKrJLRQ&t=235s
- https://memodayoungee.tistory.com/151


  







