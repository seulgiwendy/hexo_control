---
title: 스프링 앱을 센트리로 감시하자. (+스프링에서의 커스텀 예외처리 약간)
date: 2017-12-08 14:19:48
tags:
    - Java
    - Spring
    - Sentry
    - Monitoring
---

### 쎈트리(Sentry)로 서버를 감시해보자.

미천한 코딩 실력과 다양한 예외 상황으로 인해 우리의 스프링 부트 서버는 항상 다양한 Exception들을 뱉어낸다. 아 사실 Exception의 발생과 코딩 실력과는 별 관련이 없을 듯하다. 오히려 적절한 시점에서 Exception을 던져 예외 상황을 맞이한 서버가 뻗지 않도록 구현하는 것은 고급 개발스킬에 해당하고, 이런 예외상황을 모두 상상해볼 수 있는 것도 개발자의 역량이라고 본다. 

나는 그동안 다양한 종류의 Exception들을 System.out 으로 처리했었고 slf4j와 같은 로깅 프레임워크를 학습한 후엔 log.debug()나 log.error()를 불러내 콘솔에 현시하는 방법을 채택했었다. 그러나 이런 방법은 Exception 상황이 발생할 때에 콘솔을 주시하고 있어야 한다는 문제점이 있고 나중에 Exception에 대한 통계 정보를 만들어내는 것도 어려움이 따른다. 

뭔가 직관적인 대시보드를 제공하면서 실시간 에러 수집이 가능한 프레임웍이나 서비스를 찾던 도중 [Sentry](https://sentry.io)가 눈에 들어왔고 아래 삽질기를 통해 서버사이드에서의 도입 과정을 간략히 설명해 보고자 한다. 

*센트리 회원가입, DSN 정보 취득 등 Trivial한 부분은 이 글에서 다루지 않는다.*

#### 스프링의 Custom Exception Handling에 대해 알아보자.

다들 알다시피 스프링을 쓰다보면 예외 상황이 참 많이 난다. 기본적인 경우에 이런 Exception들은 "Whitelabel Error Page"라는 멋대가리 없는 페이지로 표시된다. 

스프링은 이런 예외 상황 하에서 개발자의 책임으로 적절한 에러 표시를 띄워주기 위해 Controller, Service 등 DispatcherServlet 아래에서 일어나는 모든 예외 상황을 처리해주는 HandlerExceptionResolver 인터페이스를 두고 있다. 

만약 어떠한 방식으로든 이 Exception Resolver가 패키지 안에서 구현된다면 당신의 서버가 내뿜는 예외 상황들의 처리가 이 Resolver 구현체에서 전담될 것이며, 동시에 Ordered 인터페이스를 구현해 각 Exception Resolver간의 순위를 정해줄 수 있다. 

```
public Class ShitExceptionResolver implements HandlerExceptionResolver, Ordered {

    @Override
    public int getOrder() {
        //이 메소드는 Ordered 인터페이스의 구현체이다. 예외 처리기의 실행 순위를 명시해주는 것이다. 

        return Integer.MIN_VALUE; //숫자가 작을 수록 우선권을 갖는다. 
    }

    @Override
    public ModelAndView resolveException(HttpServletRequest req, HttpServletResponse res, Object handler, Exception e) {
        //이 메소드는 HandlerExceptionResolver를 구현한 것이다.

        (...에러를 모델에 추가하는 로직은 알아서 잘 구현하자.)

        return "error"; //template 폴더에서 error.html을 찾아서 돌려주면 된다.
    }
}
```

상기한 코드에서 getOrder()의 구현을 통해 최상위 에러 처리 로직으로 스프링 빈에 등록되었으므로, 이제 서버가 뱉어내는 모든 예외는 최우선적으로 위의 Resolver가 처리하게 된다. 

**뭔가 감이 오지 않는가?** 우리는 이제 저 구현체가 모든 에러 정보를 흡수한 다음, 우리의 센트리 계정으로 중계해 주게끔 구현할 것이다. 

#### Dependency 추가 및 터닦기 작업 

센트리 포탑을 설치하기 위한 기초 터닦기 공사를 해보자. 

Maven(당연한 얘기지만 Gradle도 상관 없다) dependency를 아래와 같이 추가해 준다. 

```
    <dependency>
            <groupId>io.sentry</groupId>
            <artifactId>sentry</artifactId>
            <version>1.6.3</version>
        </dependency>
```

이렇게 추가된 클래스 Sentry를 통해 static 메소드 몇 가지를 쓰게 된다. 

그리고 Component Scan이 이뤄질 수 있는 위치에 아래와 같이 Bean을 선언해주자. (이게 필요한 과정인지는 모르겠으나 일단 적어둔다. 경험자 분들의 태클 환영)

```
@Bean
public HandlerExceptionResolver sentryExceptionResolver() {
    return new SentryExceptionResolver();
}
```

#### Exception Resolver 구현, 에러 전송

아래와 같이 Exception Resolver를 구현해 지긋지긋한 Exception들을 Sentry로 보내버리자. 

```
public class SentryExceptionResolver implements HandlerExceptionResolver, Ordered {

    @Override
    public int getOrder() {
        return Integer.MIN_VALUE;
    }

    @Override
    public ModelAndView resolveException(HttpServletRequest req, HttpServletResponse res, Object o, Exception e) {
        
        Sentry.init(YOUR_DSN); //여기서 센트리를 초기화시킨다. 
        Sentry.capture(e); //여기서 Exception을 센트리로 보내버린다.

        return null;
    }
}
```

**null을 리턴하는 이유**는 이 Resolver는 단순히 예외를 잡아다가 센트리로 보내버리는 중계자 역할만을 할 뿐, 의미있는 예외 처리를 수행하고 있지 않기 때문이다. resolveException() 메소드의 리턴 타입이 ModelAndView임을 주목하자. 이 메소드는 Exception을 적절히 다뤄 *사용자가 의미있는 정보를 UI적으로 취득할 수 있게 해주는 것*에 의미를 두는 메소드이다. 

SentryExceptionResolver는 그와 같은 기능을 못 해주고 있으므로 null을 리턴해 차순위 Exception Resolver가 본연의 기능을 수행할 수 있도록 양보해 주는 것이 맞다. 

아, 물론 원한다면 SentryExceptionResolver가 view 기능도 수행하게끔 구현해도 되지만 **비추한다.** 단일 책임의 원칙에 부합하지 않는 모습이라고 본다. 

#### 끝났다.

당신의 서버는 이제 당신의 Sentry 계정으로 예쁘게 Exception 정보를 보내줄 것이다. 

![참 쉽죠?](https://s3.ap-northeast-2.amazonaws.com/seulgiwendy.github.io-static/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2017-12-08+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+3.04.11.png "대시보드 화면")

**어때요, 참 쉽죠?**

이제 Exception을 세련되게 관리해 봅시다. 