---
title: HTTP에 대해 이거저거 알아보다.
date: 2017-10-24 11:56:18
tags:
    - HTTP
    - JAVA_BACKEND
    - Web
---

### 2017.10.24 (화) HTTP에 대해 포비와 이거저거 이야기 나누다. 

포비와 허심탄회하게 HTTP에 대해 이야기를 나눴습니다. 대화 내용을 정리하면서 배운 내용도 같이 정리해봅니다.
볼드 : 학생들, 그냥 : 포비




포비: 여러분이 이제 HTTP 웹 서버를 구현하면서 생긴 다양한 궁금증을 공유해봤으면 좋겠다. 

#### 궁금증 1 : 302 Redirection과 301 Redirection은 어떤 차이가 있는가? 

**워드프레스를 운영하면서 습관적으로 301 리디렉션을 해 왔다. *이 방법이 Google등에서 SEO에 더 잘 걸리기 때문에 그렇게 해 왔다.* 그러나 우리의 이번 프로젝트에서는 한 번도 301 리디렉션을 한 적이 없다. 차이가 궁금하다.**

포비: 302와 301 코드의 별명을 살펴보는 것이 좋겠다. 302는 **일시적 변경**이다. 반면 301은 **영구 변경**이다. 큰 차이가 있다.
기본적으로 301 코드를 받으면 클라이언트는 *새로운 주소를 기억한다.* 이렇게 되면 추후에 서비스가 다른 곳으로 옮겨갔을 때 서버가 캐시를 지워줄 수 없다. 이는 좋지 못한 사용자 경험일 것이다. 

301의 사용보다는 302를 사용하는 것이 대부분의 경우 안전한 선택이다. 

#### 궁금증 2: Session과 Cookie에 대한 이야기 

**세션과 쿠키의 개념이 혼동되는 것이 많다. 또 세션도 쿠키를 사용한다고 하는데 어떤 차이를 가지는지 궁금하다. 또 세션의 키는 누가 관리하는 것인지 알고싶다.**

포비: 세션이 쿠키를 사용한다는 것은 맞는 말이다. 정확히 말하면 **서버가 발급한 Session ID가 클라이언트의 쿠키에 박히는 것이다.** 서버도 Session ID를 저장하고 있다. *클라이언트가 Request Header에서 전송하는 쿠키에 저장된 Session ID를 통해 사용자 정보를 식별하는 것이 HTTP Session의 기본 개념이다.* 

Map구조로 된 저장소를 생각해보면 좋겠다. 여러분이 Session 정보를 쓰고자 할 때 Map에 .get()을 Session ID를 넣고 실행시키는 것이다. 즉, Session ID를 Key로 갖고 여러분이 저장시키고자 하는 정보를 POJO로 Value에 넣는 것이다. 이 때 Session은 무조건 하나의 key만 가지므로 Value에는 또 다른 형태의 Map이 들어갈 것이다. 

또 하나 오해하면 안 되는 점이 *HttpSession은 Servlet의 스펙이지 Spring의 기술이 아니다.* Spring이 아니어도 Session 관리 할 수 있다. 

##### 세션에 관한 심화 과정 

포비: 그런데 우리가 세션을 사용할 때에 이슈가 몇 가지 있다. 우선 로드밸런싱의 문제다. 우리가 로드밸런싱을 위해 WAS 상단에 Nginx, Apache와 같은 웹 서버를 두고 그 밑에 Tomcat 등 WAS를 둔다고 가정해 보자. 

0번 서버가 갖고 있는 Session ID를 1번 서버와 2번 서버는 갖고 있지 않을 것이다. 만약 로드밸런싱을 통해 특정 사용자의 요청이 0번 서버에서 처리되다가 1번 서버로 중계될 경우, *사용자는 로그인이 저절로 풀려버리는 기분나쁜 경험을 하게 된다.* 이는 기술적으로 해결해 줘야 할 이슈이다. 

이 이슈를 해결하기 위해 크게 세 가지 방법이 있다. **Sticky Session, Global Session 그리고 Session Clustering이 그것이다.** 

* Sticky Session
    * 한 사용자가 서버에서 세션 정보를 한 번 획득하면 계속 그 서버로 연결되게 조정해주는 것이다. HAProxy와 같은 오픈소스 소프트웨어로 구현할 수도 있고, 네트워크 장비로 구현하려면 OSI 7번째 계층에 접근할 수 있는 L7 장비를 사야 한다. *비싸다.*
* Global Session
    * n대의 서버 외부에 Session DB를 둔 후 같은 세션 풀을 공유하는 것이다. 
* Session Clustering
    * n대의 서버가 모두 같은 세션 정보를 공유하는 것이다. 즉 동일한 세션의 복사본이 n부 만들어지는 것이다. 

상황에 따라 적합한 해결책은 다를 수 있고 가용한 자원을 잘 따져봐야 할 것이다. 

#### 궁금증 3: 웹 서비스의 속도는 무엇이 결정하는가? 

포비: 우리가 웹 페이지 하나를 열 때 **한 번의 요청만 들어가는 것이 아니라는 점을 알아야 한다.** 사용자가 보는 건 하나의 페이지지만 페이지는 수많은 GET 요청을 날리고 있다. 

우리가 보는 수많은 .js와 .css가 계속 로드된다. *이들이 접속할 때마다 항상 새로 로드되어야 할까?* 이 자원들이 자주 바뀌는 자원인지는 고민해 볼 필요 있다. 바뀌지 않으면 서버에서 새로 이 자원들을 땡겨올 필요가 있을까? 

HTTP 헤더를 통해 이런 자원들의 생명주기를 결정할 수 있다. 클라이언트가 얼마나 이 자원들을 들고있을지 정해주는 것이 좋겠다. 자주 변경되지 않는 자원이라면 생명주기를 엄청나게 길게 줘도 상관이 없을 것이다. 또 base64 인코딩을 쓰거나 .js, .css 자원을 Minify, Uglyfy하는 옵션도 고려해 볼 만 하겠다. 

#### 궁금증 4: 스레드는 무한히 만들어지는가? 

포비: 여러분이 지금 만든 웹 서버는 사용자 요청이 1개 만들어질 때마다 스레드를 한 개씩 생성한다. 이게 좋은 방식일까? 

**우리들: 좋은 방식이 아닌 것 같다. 스레드는 많이 돌면 전환 비용(Context Switching)이 발생한다.**

*(전환 비용에 대해서는 호눅스가 지난 코스 CS 시간에 다룬 적이 있다. 스레드가 자신의 작업 내용을 레지스터에 넣고 CPU를 점유하는 데까지 걸리는 시간이 발생하는데 이것이 콘텍스트 스위칭이다.)* 

포비: 맞다. 그래서 Tomcat은 무한히 스레드를 만들지 않는다. 요청이 들어왔을 때 스레드를 만드는 것도 아니다.
**미리 스레드를 만들어뒀다가 하나씩 꺼내 쓰는 것이다.** 이것을 *스레드 풀(Thread Pool)* 이라고 한다.

자, 그러면 몇 개를 만들어서 가지고 있는 걸까? 

**우리들: 100개? 1024개? 500개?**

포비: 너무 많은 것은 좋지 않다고 했다. 기본적으로 Tomcat은 **200개의 스레드**를 만들어 가지고 있는다. 
이 스레드보다 많은 접속이 들어오면 어떻게 해야 할까. 요청을 Queue에 담아 잠시 기다리게 해야 한다. 

이 Queue의 용량도 초과되면 그 때 비로소 Tomcat은 **괴상한 메시지를 노출하며 뻗게 된다.** 

접속 부하가 많이 걸리는 웹 서비스는 Queue만을 관리하는 별도의 서버를 둘 필요가 있겠다. *그래서 명절에 기차표 살 때 새로고침을 누르면 안 되는 것이다. 새로고침을 누르면 줄에서 빠져 가장 뒤로 간다.* 이는 Queue가 Socket을 갖고 있기 때문이다. 새로고침을 누르면 Socket이 새로 만들어지고 Queue에 박혀 있던 소켓은 주인을 잃게 된다. 


#### 느낀 점

* 공부할 것이 많다. 
* OAuth에 대한 궁금증을 완전히 해소하지 못한 기분이다. OAuth 클라이언트 구현에 있어 클라이언트 사이드에서도 토큰을 갖고 있던데, 이 이유가 뭔지 궁금하다. 
* HTTP에 대한 막연함을 많이 해소한 좋은 시간이었다. 특히 Session과 Cookie가 작동하는 방식에 대한 이해를 갖게 되었다. 

