---
title: 스프링에 관한 질문들 
date: 2018-06-04 18:58:04
categories:
    - java
    - spring
---

### 스프링에 관한 질문들

*기술 면접을 2주 앞두고 면접에 필요한 내용들을 정리하며 "...질문들" 시리즈를 연재해본다.*

**스프링의 구조, 요청 / 응답**

- 스프링은 어떻게 요청을 처리하는가? 요청을 처리하는 순서대로 설명해 보아라. 

면접관님 우선 이렇게 어려운 질문을 해주신 것에 대해 제 자신을 뒤돌아보게 됩니다. 혹시나 제가 채용 과정중 맘 상하게 해드린 부분이 있다면 용서를....


Spring MVC의 관점에서 설명하겠다. Spring MVC는 **Front Controller 패턴**을 채택하고 있다. 

Front Controller란 웹 애플리케이션의 가장 앞단에 **모든 요청을 받는 서블릿**을 두고 일단 들어온 요청을 개발자가 명시한 바에 따라 분기처리해나가며 요청을 처리하는 방식이다. Spring MVC에서 Front Controller의 역할을 하는 서블릿을 **DispatcherServlet** 이라 칭한다. 

DispatcherServlet은 한개일 수도 있고 **복수개일 수도 있다.** 단 DispatcherServlet을 분할하여 관리할 수 있게 한 것은 Legacy JSP와의 호환을 위해 그렇게 남겨둔 것으로 **한개의 DispatcherServlet이 모든 요청을 받아들인 후 IoC Container에 진입해 분기처리할 수 있는데 굳이 복수개의 서블릿을 운용할 필요는 없다.** 

DispatcherServlet 앞에 Filter라는 것이 있다. Filter는 요청을 말그대로 필터링하면서 HTTP request header 등 정보를 읽어들여 DispatcherServlet에 진입하기 전 필요한 처리를 한다. 

* *꼬리질문: Filter와 Interceptor의 차이는?* 
    - Filter는 IoC Container에서 관리되는 콩이 아니다. Filter는 DispatcherServlet보다도 앞단에 위치하는 객체로 Spring이 관리하지 않는 객체이므로 Dependency Injection 등의 구현을 할 수 없다. 
    - 그대신 Filter를 Spring Config class에서 등록시키고, 이 때 Autowired된 객체의 참조를 생성자로 전달할 수는 있다. 실제로 이런 구현을 많이 한다. 예를 들어 SecurityConfig.java에서 인증에 필요한 객체들을 bean으로 받아놓은 후 filter에 참조를 넘겨줄 수 있다. 
    - Interceptor는 스프링이 관리하는 콩이다. 

일단 DispatcherServlet에 진입하고 나면 적합한 컨트롤러를 찾아 매핑시켜줘야 한다. 최근의 Spring MVC에서는 AnnotationMethodHandlerAdapter가 사용된다. @RestController나 @Controller로 애노테이션 된 클래스 아래에 있는 @GetMapping, @PostMapping 등 메소드들이 스캔의 대상이 된다. 

HandlerAdapter는 간단한 인터페이스로 handle() 메소드 하나가 전부다. 적합한 컨트롤러를 찾아 HttpServletRequest를 전달하고 응답을 돌려주는 역할을 한다. 

Spring MVC에서 사용할 수 있는 컨트롤러의 타입에는 제한이 없다. 개발자가 마음껏 정의해서 쓸 수 있다. 이를 가능하게 해주는 것이 HandlerAdapter이다. 컨트롤러의 메소드에서 파라메터를 전달받을 수 있다. 이 파라메터를 전달해주는 역할은 ArgumentResolver가 수행한다. 

기본 등록되어있는 ArgumentResolver 이외의 다른 객체를 파라메터로 사용하고 싶다면 custom으로 구현해주면 되겠다. 

HandlerAdapter의 handle() 메소드가 갖는 리턴 타입은 ModelAndView다. HTTP Body에 직접 접근해 응답을 기록하는 @ResponseBody 애노테이션이 붙어있지 않는 이상 모든 컨트롤러는 ModelAndView로 응답을 전달해야 하고, ViewResolver가 그 역할을 수행해준다. 이때 property에 기록된 template suffix 등이 적용된다. 

@ResponseBody나 @RestController의 경우 ModelAndView 타입을 리턴하는 ViewResolver 대신 MessageConverter가 작동한다. MessageConverter는 컨트롤러의 메소드가 리턴한 객체를 전략에 따라 적절한 message로 변환하여 HTTP body에 직접 내용을 기록한다. 

대표적인 것으로 Jackson2HttpMessageConverter.java 가 있다. 이 객체는 컨트롤러가 리턴한 객체를 com.fasterxml.jackson.objectMapper() 의 writeValueAsString() 메소드에 넣고 스트링으로 객체를 변환한 후 해당 스트링을 http body에 기록하게 된다. 

ModelAndView 타입이건 ResponseBody에 대한 응답이건 응답이 기록되게 되면 Spring MVC의 요청 처리 사이클이 완료된 것이다. 

- *꼬리질문: DispatcherServlet은 IoC Container에 바로 접근해서 Spring bean을 사용할 수 있는가? 

    - 반은 맞고 반은 틀림. IoC Container의 구현체인 Web application context는 root application context와 servlet application context로 구분됨. Servlet Web ApplicationContext는 controller 등 handleradapter와 requesthandler를 위한 Spring Bean만 갖고 있음. Servlet Web ApplicationContext는 복수개일 수 있음. 
    - Root Web ApplicationContext는 하나의 프로젝트 당 하나임. 복수개로 등록된 Servlet Web ApplicationContext가 하나의 Root Web ApplicationContext를 바라보며 DAO, service 계층의 스프링 빈 등 의존성을 이용해 요청을 처리하는 것임
    - DispatcherServlet 한개당 Servlet Web ApplicationContext는 하나여야 함. 

**Spring IoC Container**

- IoC Container라는 언급을 자주 하셨는데 이는 무엇인가? 

IoC Container는 스프링 그 자체다. 

DI Container라고도 하고 이걸 그냥 스프링이라고 부르는 개발자들도 있다. 혹은 Singleton Container라고 불리기도 한다. 

핵심은 **싱글톤 보일러플레이트 없이** 객체들의 싱글톤 상태를 유지시켜주는 것이다. 이를 위해 스프링은 객체들을 직접 생성해 컨테이너 안에서 그 생명주기를 관리하며 싱글톤 상태를 깨지 않는 관리를 하게 된다. 

다른 객체에 대한 의존성 주입이 필요하다면 IoC Container가 직접 주입해준다. 여기서 제어권의 역전이 발생한다. 객체는 특정 객체가 아닌 타입에 의존하면서 실제 구현체에 대해서는 모르는 상태를 유지하는 것이다. 

스프링의 ApplicationContext는 간단한 인터페이스로써 BeanFactory를 상속하고 있다. .getBean() 을 통해 Bean 객체를 요청받으며 만약 싱글톤 레지스트리에서 보관하고 있지 않는 객체인 경우에는 BeanFactory를 통해 새로운 객체를 만들어서 돌려준다. 이 때도 객체는 싱글톤 레지스트리에 등록시키며 앞으로 다른 요청이 있을 경우 싱글톤 레지스트리에서 빼서 갖다준다. 새로운 객체를 두 번 만들지 않는다. 

- *꼬리질문: Spring이 이런 구현을 가져가는 이유는 뭐라고 생각하는가?*
    - 아무리 JVM이 현대화되고 최적화됐지만 객체의 생성은 약간의 오버헤드를 가져오는 일임에는 틀림이 없다. 
    - 접속이 새로 들어와 스레드가 생길 때마다 새로운 서비스 객체, 각종 컴포넌트 객체가 새로 생긴다면 JVM의 객체 생성과 GC 메카니즘에는 상당한 부하가 걸릴 것이다. 
    - 멤버 변수를 가질 필요가 없고 비즈니스 로직을 수행하는 객체라면 Bean으로 관리하는 것을 고려해야 한다. 

스프링은 ApplicationContext로 객체를 관리하면서 다양한 부가기능을 구현할 수 있게 도와준다. Spring AOP가 대표적인 사례로 애노테이션을 통해 객체에 부가기능을 추가할 수 있는 것은 결국 Spring의 ApplicationContext가 런타임 시에 객체를 만들어 싱글톤 레지스트리에 등록하는 과정 중에서 프록시 객체의 생성과 같은 객체 내용/역할의 변조가 가능하기 때문이다. 

- *꼬리질문: Spring AOP의 작동 원리에 대해 설명해 보아라.*
    - Spring AOP를 가능케 해주는 두 가지 방법, 그리고 크게 세 가지의 Framework support가 있다. 
    - 먼저 Runtime Weaving 부터 살펴보면 JDK Dynamic Proxy와 CGLib이 있다. 
    - JDK Dynamic Proxy는 decorator 패턴을 따르는 것으로 proxy 객체를 형성해 부가 기능의 구현을 전담시킨 후, 원래 기능의 수행은 원 객체에 위임시키는 구현 방식이다. 
    - JDK Dynamic Proxy의 proxy 객체에는 .invoke() 메소드가 존재한다. .invoke() 메소드는 InvocationHandler의 구현체에 존재하는 메소드이다. 
    - JDK Dynamic Proxy는 인터페이스에 대해서만 weaving할 수 있다.
    - concrete class에 대한 weaving은 CGLib을 사용한다. CGLib은 원 객체를 상속해 새로운 객체를 구현하는 방법으로 weaving을 실시하고, class와 method에 대한 overriding이 수반되는 방식이다. 따라서 코틀린과 함께 이용할때는 open 예약어를 통해 상속을 오픈시켜줘야 한다. 
    - AspectJ를 응용한 Compile Time Weaving도 가능하다. AspectJ를 사용하는 경우 Load Time Weaving도 가능하다. 
    - AspectJ는 실제 바이트 코드에 대한 변조이므로 제약조건이 가장 적고 성능도 가장 좋게 나온다. 
    - 주의해야 할 점: Spring AOP에서 @AspectJ 애노테이션을 사용하는 것은 AspectJ를 통한 compile time weaving을 수행하는 것이 아니다. JDK Dynamic Proxy / CGLib 기반 Runtime weaving 하에서 AspectJ의 **문법만을 갖다 쓰는 것** 에 불과하다. 

- *꼬리질문2: AOP 기반 Transaction 관리에 대해 설명해 보아라.* 
    - javax.persistence.Transactional 을 붙이면 트랜잭션 부가 기능 프록시 객체가 서비스 클래스에 붙게 된다. 
    - Transaction을 시작하고 소멸시키는 생명주기가 프록시 객체 안에서 이뤄지게 된다. 
    - 클래스 레벨로 설정할 수도 있고, 메소드 레벨로 설정할 수도 있지만 중요한 사실은 **클래스 내부 호출에 대해서는 의도대로 작동하지 않는다** 는 점이다. 
    - 프록시 객체가 받은 요청에 대해서만 transaction이 사용될 것이다. 즉, public 하게 선언된 메소드를 통해 **작업이 직접 호출되어** 수행된 경우만 프록시 객체가 transaction 관리를 해줄 수 있다. 내부 호출에 대해서는 이미 동작이 원래 객체에 위임된 시점 이후의 일이기 때문에 프록시가 트랜잭션 관리를 해줄 수 없다. 

- ApplicationContext가 객체를 만들고 보관하는 과정에 대해 자세히 설명해보아라. 

먼저 ComponentScan 애노테이션에서부터 모든 과정이 시작된다. 부트 환경에서라면 @SpringBootApplication이 그 시작이 되기도 한다. 

스프링은 처음 구동될 때 @Bean, @Component, @Service 등의 애노테이션이 붙어있는, 즉 자기가 관리해야 할 클래스를 모두 스캔한다. 이 때 스캔의 범위는 개발자가 적절하게 부여해줄 수 있다. 

만약 instantiate되어 관리돼야 할 클래스가 다른 빈에 다시 의존하고 있다면 재귀적으로 해당 빈에 대한 instantiation도 수행한다. 이런 식으로 전체 패키지(혹은 개발자가 스캔하라고 지시한 패키지)에 대한 스캔을 수행하며 아무런 의존성도 업는 객체를 발견할때까지 재귀호출을 계속한다. 

- *꼬리질문: Spring은 어떻게 자신이 관리할 빈인지 알아보는 것인가?*
    - 애노테이션으로 관리된다. 혹은 .xml등 외부 설정 파일을 통해 관리해줄 수도 있다. 중요한 사실은 스프링은 이런 설정 관리를 위해 **특정한 파일 형식이나 표현방법을 강제하진 않는다** 는 점이다. 
    - 클래스에 붙은 애노테이션을 검색하고 해석하기 위해 Reflection를 사용한다. reflection 작업에 대한 최적화를 위해 스프링 개발자는 JVM 튜닝 등 다양한 전략의 도입을 고려할 필요도 있다. 

- *꼬리질문2: 어떻게 하면 리플렉션을 최적화할 수 있을까?*
    - Reflection이 들여다보는 메모리 영역에 대한 최적화일 것이다. 리플렉션이 들여다보는 정보는 class area에 저장된다. 
    - JDK 1.7까지 이 영역은 Permanent Generation(이른바 PermGen)에 위치했다. JDK1.8 환경부터 이 영역은 metaspace로 이동했다. 
    - 솔직히 저도 잘 모르겠는데 입사하면 열심히 배우겠다. 

- *꼬리질문3: 애플리케이션 콘텍스트가 초기화되는 과정 중 일어날 수 있는 문제들은 어떤 게 있는가?*
    - 가장 흔한 것으로는 의존해야 할 객체가 없는 경우, 즉 @Autowired를 통해 주입받고자 하는 객체나 설정 정보가 주입되지 않는 경우다. 
    - properties 파일에 있어야 할 설정 정보가 부재하거나 interface만 선언해 놓고 타입에 의존한 다음 실제 구현체를 만들어주지 않은 경우가 여기에 해당한다. 
    - 두 번째로 흔한 오류는 circular reference 즉 순환 참조의 오류다. 같은 클래스에서 의존하고 있는 두 가지 이상의 객체가 서로에 대해 의존하고 있는 경우 발생한다. 
    - 이 경우 @Bean 이 선언되어 있는 설정 클래스를 분리해주거나 @Primary 애노테이션 등을 통해 instantiation의 우선순위를 바꿔주면 잘 해결된다. 
    - 정말 드문 경우가 있는데 Servlet Web ApplicationContext에 있는 객체에 대해 RootApplicationContext가 관리하는 객체에서 의존하는 경우다. 정말 드문 경우고 최악의 경우라고 볼 수 있다. 
    - 의존성은 위에서 아래로 흐르는 것이 가장 좋다. 의존성 체인의 가장 밑단에 있어야 할 컨트롤러 등 프레젠테이션 레이어의 클래스들은 주입받으면 안된다. 
    - 서비스는 컨트롤러를 주입받을 수 없다. 

**Spring Security**

- 핵데이 프로젝트에서 스프링 시큐리티를 사용하셨는데 어떤 흐름으로 인증이 처리되는지 간단히 설명해 보아라. 

인증이 처리되는 과정은 크게 세 파트로 나눌 수 있겠다. 요청 - 인증 - 인가 가 그것이다. 

먼저 요청이 들어오면 DispatcherServlet 앞단에 위치하고 있는 javax.servlet.Filter 구현체들이 요청을 반갑게 맞아줄 것이다. 

RequestMatcher 등을 통해 이 요청이 인증받아야할 요청인지, 혹은 로그인을 위한 요청인지를 판별하게 된다. 이때 필터 객체는 이미 구현되어 있는 스프링 시큐리티 관련 객체들에 대한 참조를 갖고 있는 상황이다. 

일단 필터가 요청에 대한 인증이 필요하다고 판단하면 AuthenticationManager를 호출하여 인증 절차를 시작한다. 이 때 요청을 그대로 매니저로 넘기는 것은 아니고 요청에 필요한 정보만 쏙 빼서 Authentication 클래스의 서브클래스들 (대표적인 것으로, UsernamePasswordAuthenticationToken)에 정보를 담아 매니저로 보내야 한다. 

AuthenticationManager는 별거 아니고 거대한 주머니에 불과하다. AuthenticationProvider를 들고 있는 주머니이다. .authenticate() 메소드를 통해 인증 요청이 접수되면 AuthenticationManager는 자기의 주머니에 담겨있는 AuthenticationProvider를 하나 하나 훑어보며 어떤 공급자가 이 요청을 잘 처리해줄 수 있을지를 고민한다. 

이제 역할을 AuthenticationProvider에게 넘어왔다. 매니저가 문을 두드리며 "이 요청 네 것 맞니?" 라고 물어볼때 자신의 요청이다싶으면 true를 반환하고, 인증을 수행하기 시작한다. 

모든 인증정보가 명확해 인증해줘도 되겠다는 판단이 들면 AuthenticationProvider는 마찬가지로 Authentication의 서브클래스인 인증 객체를 리턴한다.

리턴할 인증 정보는 Authentication의 서브클래스라면 무엇이든 좋지만 UserDetails 인터페이스의 구현체들이 애용된다. Spring Security는 기본 구현체로 org.springframework.security.core.User 클래스를 제공한다. 이 때 User 정보를 만들기 위해 UserDetailsService 인터페이스를 구현하는 경우도 있다. 

모든 인증 절차가 성공적이었다면 filter가 그 결과를 돌려받는다. .successfulAuthentication() 메소드가 호출되는데 이 때 인증 정보를 기억시켜줘야 같은 스레드 내에서 요청 정보가 공유될 수 있다. 새로운 SecurityContext를 만들고 SecurityContextHolder에 집어넣어 보관한다. S..Holder는 ThreadLocal 객체를 통해 인증 정보를 관리하기때문에 같은 요청에 대해서는 계속 인증 정보가 보관된다. 

* 하나의 요청이 두 개 이상의 스레드를 만드는 경우 애노테이션을 통해 인증정보 관리 전략을 갈아끼울 수 있다. 

    - *꼬리질문: 왜 스프링은 프로바이더를 통해 인증 과정을 관리하는가?*
        - 스프링이 지향하는 핵심 가치 중 하나는 **확장성** 과 **customizable 함(?)** 이라고 생각한다. 
        - 개발자의 의도에 따라 얼마든지 인증 과정을 추가할 수도, 혹은 변경할 수도 있어야 한다. 
        - 같은 웹 애플리케이션 콘텍스트 안에서 여러가지의 인증방법을 가져가야 할 수도 있다. 예를 들어 단순한 로그인 창을 통한 로그인 요청이라면 UsernamePasswordAuthenticationToken 을 사용하고 UserDetailsService에 의존하는 인증 프로바이더로 충분하지만, 저의 프로젝트와 같이 JWT라도 도입할라 치면 만약 이렇게 확장성없는 구조로 구현됐다면 일일이 하드 코딩을 통해 변경 내용을 도입했어야 할 것이다. 

    - *꼬리질문: 인증 전, 후 를 가르는 기준은?*
        - 스프링은 "완전한 정보를 갖고 있는 인증 객체(fully-populated authentication principal)"라면 인증 절차가 끝난 객체로 본다. 
        - 완전한 정보를 가르는 기준은 대부분의 경우에 권한을 들고 있는지, 아닌지로 구분된다. 
        - 일테면 UsernamePasswordAuthenticationToken의 경우 두 가지의 생성자를 갖고 있다. 하나는 유저네임과 비번만을 파람으로 받지만 두 번째의 경우 GrantedAuthority의 서브클래스를 가진 컬렉션을 추가로 전달해야 한다. 
        - ```Collection<? extends GrantedAuthority>```를 들고 있는 인증 객체는 .isAuthenticated() 메소드에서 true를 리턴할 것이다. 

- PreAuthorize, PostAuthorize는 뭔가? global method security 라는 놈도 붙어있던데.... 
    - AOP기반 인증 처리기다. 
    - Controller의 메소드에 이 애노테이션이 달려있으면 AOP를 통한 부가기능 추가 메카니즘으로 인증 로직을 스프링이 자동으로 구현해준다. 
    - PreAuthorize는 **메소드 진입 전** 인증 정보를 평가하고 진입을 통제한다, PostAuthorize는 그 반대다. 
    - SecurityContextHolder가 관리하는 SecurityContext, 그 속의 Authentication Principal을 통해 권한 정보를 fetch 하고 이 메소드에 접근 인가된 사용자인지를 알아본다. 




