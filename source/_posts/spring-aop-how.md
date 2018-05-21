---
title: 스프링 AOP는 어떻게 의도대로 작동하는가
date: 2018-05-02 12:26:36
categories:
    - Spring_Framework
    - Java
---

### 스프링 AOP는 어떻게 의도대로 작동하는가

스프링이 거대한 DI(Dependency Injection, 종속성 주입)컨테이너임은 누구나 아는 사실이다. 그리고 우리는 이런 컨테이너의 도움을 받아 비즈니스 로직에 집중하고, POJO만으로 서비스를 개발할 수 있음에 하루 하루 감사함을 느끼며 살고 있다. 

그러나 우리가 DI의 메카니즘 자체에 대해 고민해볼 일은 많지 않은 것이 사실이다. 단순히 `@Autowired`를 달면 알아서 종속성이 잘 주입되어 의도대로 작동하는 것을 보며 감탄하거나 무덤덤해 할 뿐 그 이상의 느낌을 갖지는 못하는 것 같다.

최근 프로젝트에서 AOP를 적용해볼까 고민하던 차에 AOP의 구현이 뭔가 장황하게 느껴지고 앞뒤가 맞지 않는듯한 느낌이 들어 학습에 어려움을 겪은 적이 있었다. 이 어려움은 Spring의 애플리케이션 콘텍스트가 사실은 거대한 DI 콘테이너에 불과하다는 것을 상기하고, DI 개념에 대한 기초 학습부터 다시 진행하니 조금씩 해결됐다. 

이번 글을 통해 스프링의 종속성 주입이 어떻게 이뤄지는지를 다시 한 번 복습해 보고, AOP의 핵심 개념에 대해 알아보는 시간을 갖고자 한다. 

**종속성 주입**

우리는 걸그룹의 인사 구호를 외쳐주는 클래스를 작성할 것이다. 걸그룹들은 저마다의 [독특한 인사 구호](https://www.youtube.com/watch?v=6mS2iZa1QMA) 를 갖고 있다. 

단순하게 생각해보면 이런 식의 구현을 가져갈 수 있겠다.

```java
public class GirlGroupHello {
    
    private static final String REDVELVET_HELLO = "해피니스! 안녕하세요 레드벨벳입니다!"
    private static final String TWICE_HELLO = "원 인어 밀리언!"

    public String sayHi(String groupName) {
        
        switch(groupName) {
            case "레드벨벳":
                return REDVELVET_HELLO;

            case "트와이스":
                return TWICE_HELLO;
        }
    }
}
```

단순하고 좋아보이는 구현이지만 몇 가지 문제점이 눈에 들어온다.

- 객체지향 설계의 안티패턴(anti pattern)이라고 잘 알려져있는 `switch-case` 가 핵심 로직으로 구현돼있다. 
- 차후 확장의 여지가 적다. *세상에는 레벨 트와 말고도 걸그룹이 많잖아?*

이런 방식이면 어떨까?

```java
public class GirlGroupHello {
    
    private HelloWords helloWords;

    public GirlGroupHello(HelloWords helloWords) {
        this.helloWords = helloWords;
    }

    public String sayHi() {
        return helloWords.sayHi();
    }
}

public interface HelloWords {
    
    String sayHi();
}

public class RedVelvetHelloWords implements HelloWords {

    @Override
    public String sayHi() {
        return "해피니스! 안녕하세요 레드벨벳입니다!";
    }
}
```

이제 `GirlGroupHello` 객체는 인사구호를 만들어주는 로직에 대한 통제권을 완전히 잃어버렸다. 대신, 객체가 생성될 때 주입받은 종속성이 만들어주는 인사구호를 그냥 전달해주는 역할만을 받게 되었다.

객체가 자신이 수행할 동작을 정의하지 않고 주입받은 종속성을 통해 자신의 행동을 통제받는 **제어권의 역전(IoC)** 이 일어난 것이다.

개발자는 이제 새로운 걸그룹이 나올 때마다 스파게티같이 늘어진 스위치 케이스 문을 수정할 생각에 밤잠을 설치지 않아도 된다. 그가 할 일은 `HelloWords`를 구현한 새로운 클래스를 작성해 적절하게 주입시켜주는 것 뿐이다. 

**새로운 요구사항의 등장**

SM엔터테인먼트가 주 52시간 근로 규정을 준수하는지 알아보기 위해, 레드벨벳이 인사구호를 외칠 때마다 타임스탬프를 기록하는 요구사항을 구현해야 한다고 가정해보자. 

가장 쉬운 것은 인터페이스를 수정하는 것이다. 

```java
public interface HelloWords {

    String sayHi();

    LocalDateTime when();
}
```

이 방식에는 다음과 같은 문제점이 있다.

- 그동안 구현했던 `HelloWords`의 구현체를 일괄적으로 수정해야 한다. 그렇지 않으면 컴파일 에러가 발생한다.
- 정식 근로계약을 맺지 않아 요구사항의 구현이 필요 없는 걸그룹들도 있다. *(더 나쁜 것 같지만 넘어가자)*

**부가기능 구현 클래스**

다음과 같이 부가기능을 구현한 뒤 원래 구현체에 동작을 위임하는 구현을 할 수 있겠다.

```java
public class TimeAddedHelloWords implements HelloWords {

    private HelloWords helloWords;

    public TimeAddedHelloWords(HelloWords helloWords) {
        this.helloWords = helloWords;
    }

    public LocalDateTime when() {
        return new LocalDateTime();
    }

    @Override
    public String sayHi() {
        return helloWords.sayHi();
    }
}
```

이처럼 종속성 주입을 적절히 잘(...) 활용하면 **원래 코드에 대한 수정 없이도** 계속 요구사항 구현을 추가해 나갈 수 있고 기능별로 구현체를 고립시킴으로써 **유닛 테스트와 유지보수도 용이해지는** 이점을 가질 수 있다. 

**AOP가 별건가?**

AOP라는 건 결국 공통의 관심사(aspect)를 추상화해 잘 보관하고 있다가 필요한 곳에 동적으로 삽입하여 부가기능을 구현해주는 방법론에 불과한 것이다. 

스프링에서 공통의 관심사를 추상화해 잘 보관하고 있는 객체를 **어드바이스(Advice)** 라고 하고, 이 부가기능이 타깃 구현체에서 적용될 시점을 **포인트컷(Pointcut)** 이라 부른다.

스프링은 AOP를 구현하기 위해 프록시 객체를 만든다. 개발자가 작성한 빈 클래스에 부가기능을 추가하기 위해, 런타임에서 동적으로 부가기능을 주입하는 프록시 객체를 만드는 것이다. 

프록시 객체를 만드는 방법에는 두 가지 방식이 사용된다.

- JDK Dynamic Proxy : `java.lang.reflect`를 사용하는 고전적 방식이다. 인터페이스에 대해서만 프록시 객체를 만들 수 있으므로 concrete class에 대해 바로 포인트컷을 준 경우에는 사용할 수 없다.

- CGLIB : Code Generator Library의 약자라고 한다. 구현 방식은 개발자가 작성한 빈 클래스를 상속받아 프록시 객체를 만드는 것이다.
    * 클래스에 대해서도 프록시 객체를 만들어줄 수 있다는 장점이 있다(상속받으니까!).
    * 때문에 Final class는 사용할 수 없다. 
    * Kotlin과 병행 사용할때는 각별한 주의를 요한다. [Kotlin은 기본적으로 모든 클래스가 final이다](https://kotlinlang.org/docs/reference/classes.html).
        * 이 문제를 해결해주는 Compiler Plugin이 있다(Spring, Spring-JPA 등). 나중에 살펴본다. 

사실 포인트컷을 정의하는 일은 꽤나 까다롭다. 포인트컷 표현식(expression language)을 써서 메소드를 정확히 정의해야하는데 다양한 인터페이스에서 공통적으로 적용되는 포인트컷이라면 번거롭기 그지 없을 것이다. 

이런 경우 커스텀 애너테이션을 붙여 포인트컷을 정의해주는 방법도 고려해볼만 하다. 

[요렇게 하면 되겠다.](http://www.baeldung.com/spring-aop-annotation)

실제 AOP를 프로젝트에 적용해 본 사례는 다음 글을 통해 공유해 보겠다. 