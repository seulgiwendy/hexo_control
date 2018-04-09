---
title: 아침 CS - (1) 이중통신 (Duplex)
date: 2018-04-09 09:13:28
categories:
    - network
    - morning_cs
    - duplex
---

### Duplex에 대하여. 

#### 사전적 정의 

이중통신(duplex) 또는 쌍방향 통신은 두 지점 사이에서 정보를 주고 받는 전자 통신 시스템을 말한다.

이중 통신을 할 때 전송 방향마다 두 개의 통신 선호를 사용하면 단순하지만 전송로를 아끼기 위해 여러 종류의 전송 방식이 쓰인다.

#### 조작적 정의 

*전화기와 무전기의 비유*

- 전화기: 상대가 떠들 때 나도 떠들 수 있음. 
    - Full-duplex임. 
    - 애초에 인간의 대화는 한쪽이 떠들 때 듣고만 있으면 성립 안함(군대에선 성립함).

- 무전기 : PTT방식 무전기는 한 사람만 떠들수 있음(두사람이 동시에 Push 하면 혼선 발생). 파워텔 제외임 ㅋㅋㅋㅋ 시바 
    - Half-duplex임.

#### 심화

흔히 컴퓨터 네트워크는 당연히 전이중(full-duplex)라고 생각하기 쉽지만 **과연 그럴까?**

모든 컴퓨터 네트워크가 전이중이라면 stateless니, 3-way handshake니 하는 개념도 창안되지 않았을 것이다. 뭔가 답답한 구석이 있어서 이런 우회적 구현을 한 것이다. 

**놀랍게도 우리가 즐겨 사용하는 L2 계층인 이더넷(Ethernet)은 반이중(half-duplex)이다.**

*(정확히 말하면 아주 제한적인 상황에서만 전이중통신을 지원한다.)*

질문 : Ethernet이 반이중이라면, TCP가 전이중일 수 있나요? 

- 먼저 두 개념이 다른 OSI 계층 상에 위치함을 알아야 함
    - Ethernet : 데이터링크 계층(L2)
    - TCP : 전송 계층 (L4)

- TCP 프로토콜은 API상으로 full-duplex이며 REQ/RES가 전환될때 통신 재설정이 일어나지 않음

- 단 상위 계층 요소(Ethernet: L2, IP : L3)가 half-duplex인 경우 하위 계층에서 full-duplex를 지원하더라도 이를 사용할 수 없음. 

- 이러한 궁금증이 생겼을 때 각 요소가 OSI 피라미드에서 어디에 위치하는지를 살펴보고, 그에 따른 상하관계를 파악하는 것이 중요

---

#### 참고 문헌

- Stack Overflow: [Is TCP bidirectional or full-duplex?](https://stackoverflow.com/questions/28494850/is-tcp-bidirectional-or-full-duplex)
- Stack Exchange: [How is TCP duplex if Ethernet is half-duplex?](https://networkengineering.stackexchange.com/questions/7451/how-is-tcp-duplex-if-ethernet-is-half-duplex)
- [전이중통신과 반이중통신](http://www.m-system.co.jp/korean/mame_k/pdf/20.pdf)



