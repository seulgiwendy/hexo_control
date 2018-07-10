---
title: 운영체제에 관한 질문들
date: 2018-06-09 17:50:36
categories:
    - operating_system
    - tech_interview
---

### 운영체제에 관한 질문들 

* 동기와 비동기의 차이점에 대해 말해 보세요.

synchronous와 asynchronous의 차이는 "반환값의 기대" 에서 발생한다고 생각합니다. system call이 발생했을 때 리턴값이 돌아올 때까지 기다리면 동기, 리턴값이 돌아올 때까지 기다리지 않고 리턴 값이 만들어졌을 때에 수행할 로직을 참조로써 전달(콜백) 하면 비동기이다. 

동기와 비동기는 각각 blocking, non-blocking일 수도 있다. synchronous non-blocking일 수도 있다(event-driven). 대표적인 사례로 node.js를 꼽을 수 있겠다.

* 
