---
title: ThreadLocal에 대한 짧은 정리
date: 2018-06-04 12:05:21
categories:
    - java
    - thread
    - concurrent
---

### Threadlocal에 대한 짧은 정리

*(진짜 짧음 주의)*

**Thread-safe하다?**

동시성 프로그래밍을 하다 보면 thread-safe, thread-unsafe에 대한 얘기를 하게 된다. 

흔히 멤버 변수가 있는 객체를 thread-unsafe, 멤버 변수가 없는 객체를 thread-safe하다고도 하기도 하고, 혹은 synchronized나 volatile과 같은 키워드를 선언해서 thread-safe함을 보장해 주기도 한다. 

같은 메모리 영역에 보관되어 있는 하나의 객체가 두 개 이상의 스레드에서 한번에 요청될 때, 한 쪽에서 읽기/쓰기 작업이 끝나지도 않았는데 다른 스레드가 객체의 정보를 요청하는 경우가 바로 **thread-unsafe** 한 상황이다. 

**Java가 thread-safe를 보장하는 방법**

두 가지 방법이 있다. 

* Synchronized: 가장 단순한 방법이다. 단순 명쾌하다. **한 스레드의 작업이 끝날때까지 다른 스레드의 진입을 막아주는 것이다.** 메소드, 혹은 변수에 선언할 수 있다. 

* Volatile: 생소한 개념일 수 있다. 조금 복잡하다. **변수의 CPU Cache 복사를 금지하는 키워드다.** 모든 작업은 RAM(혹은 어떤 형태이든지 주기억장치)에서 이뤄져야 한다. Java는 Heap 영역에 객체가 보관되기 때문에 이 경우 객체의 원자성이 보장된다. 

![volatile](http://thswave.github.io/assets/media/0308.jpg)

이런 volatile의 작동 방식을 **변수의 가시성을 보장한다** 고도 말할 수 있다. 

말 그대로다. CPU에 꽁쳐놓지 않고 모든 작업을 주기억장치에서 처리하니 어떤 스레드에서도 작업 내용을 투명하게 열람할 수 있는 것이다. 

**ThreadLocal의 역할**

다만 inter-thread operability를 보장해주는 위의 두 키워드와는 다르게 만들어진 하나의 스레드 전역에서 공유되어야 할 정보가 있다. 

이런 정보들을 thread-safe하게 구현하려면 복잡성이 수반되는데, 단순히 heap 영역에 보관해 공유하자니 thread-safe함이 무너지고, 그렇다고 매 stack마다 선언하고 해당 stack에서 쓰고 버릴 수도 없는 경우가 있기 때문이다. 

ThreadLocal을 통해 이런 문제점을 해겷할 수 있다. 

![threadlocal](http://cfs14.tistory.com/image/36/tistory/2009/02/13/15/44/499516b885314)

말하자면 로컬 변수를 로컬처럼 쓰지 않게 도와주는 방법이라고 볼 수도 있겠다. 

*쓰임새?*

하나의 스레드가 열린 상태에서 다양한 요청을 처리하고, 또 그에 따른 정보들을 공유해야 하는 상황에 대해 생각해보면 좋을 것 같다. 

가장 쉽게 생각하자면 Spring Security의 **SecurityContextHolder**가 떠오른다. 처음 DispatcherServlet이 (혹은 그 앞의 filter가) 요청을 받았을 때 새로운 스레드가 생성되고, 이때 삽입된 인증 정보는 스레드 내에서 계속 공유되며 AOP 기반 인증 처리, 유저 정보 요청 등의 작동을 수행해야 하기 때문이다. 

*주의점?*

Thread Pool을 통해 thread를 재사용할 경우 한번 쓴 ThreadLocal 객체는 .clear() 를 통해 비워줘야 한다. **그러지 않으면 다음번 해당 스레드가 사용될 때 의도한 작동을 보장할 수 없다.**

