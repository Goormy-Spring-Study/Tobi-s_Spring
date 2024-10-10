Spring Boot에서는 트랜잭션을 제공해주는 @Transactional 어노테이션이 있다.

# 트랜잭션이란? 

트랜잭션은 ACID 원칙을 따른다. 

- 원자성 
- 일관성
- 독립성 
- 지속성

그런데 이 작업들을 일일히 JDBC의 setAutocommit, commit, rollback 해주는 것은 코드가 길어지고 실수할 여지도 생긴다. @Transactional 어노테이션은 이 필요한 코드를 자동으로 생성하고 삽입해준다. 


# 동작 구조 

![](https://velog.velcdn.com/images/dlrkdus/post/8c0b73d5-7cee-4fdc-88dd-dc35723b13c9/image.png)

Spring에서 Transaction은 AOP에 의해 구현되며, 이 AOP는 프록시 객체를 통해 구현된다. 

1. @Transactional 어노테이션이 있으면 해당 빈을 상속 받은 프록시 객체를 생성한다.
프록시 객체를 생성하는 이유는 Aspect 클래스에서 제공하는 부가 기능들을 사용하기 위해서이다. 

2. 생성된 프록시 객체는 Target에 대한 호출을 가로채 Transaction Advisor에게 전달한다.

3. Transaction Advisor가 트랜잭션을 생성한다. 

4. Transactio Advisor은 커밋, 롤백 등의 트랜잭션 결과를 반환한다. 


# 특징 

- private 메소드는 상속이 불가능하므로 어노테이션을 붙여도 프록시 객체가 상속을 할 수 없어 트랜잭션이 동작하지 않는다.

- 같은 Class 내의 여러 @Transactional 호출시 최초의 Transaciton을 기준으로 동작한다.

프록시 내부에서 내부를 호출할 때는 Trnasaction과 같은 부가적인 기능이 적용되지 않는다. 
호출하려는 타겟의 프록시를 통해야 부가 기능이 적용된다. 



