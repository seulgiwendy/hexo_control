---
title: jstat으로 JVM 메모리 사용을 구경해보자. 
date: 2018-06-19 14:25:37
categories:
    - jstat
    - jvm
    - java
---

### jstat으로 JVM 메모리 사용을 구경해보자. 

JVM에 대해 학습하다보면 gc가 정말 이론대로(?) 움직이는지 궁금해질 때가 있다. 

IDE에서의 디버깅으로는 이런 메모리 정보를 직관적으로 알기 어렵다. 그래서 jstat이라는 도구가 있다. 참 좋은 도구다. 

```
jstat -옵션 -JVM pid
```

옵션에는 다양한 프로파일링 타겟이 존재한다. 

옵션명 | 특징 
--- | ---
class | 클래스 로더 동작
gc | heap 영역 전체에 대한 데이터
gcnew | New 세대에 대한 데이터

등등이 있다. 이중 내가 시도해본 것은 gcnew이다. 

![작동중인 jstat](https://scontent-icn1-1.xx.fbcdn.net/v/t1.0-9/35495645_542053236189591_3224093560965955584_n.jpg?_nc_cat=0&_nc_eui2=AeGfel3hb7tB27dSXNWkPYTKygGbG9RonYNwQChcwFYkCwwkVu0Rd2tKhi4ekYe3IaDTDBwRehHDv2O-QbOd_RBWfjdrPvQQWTbn9PmCpCmkkg&oh=db8ee0c93ee7d69d1f9948b027929526&oe=5BEB815C)

위 사진에서 맨 오른쪽 열이 gc counter, 즉 new 영역에 대한 minor GC가 발생한 숫자를 기록하고 있는 부분인데 12에서 13으로 한 개 증가하며 한 칸 왼쪽의 열이 보여주고 있는 Eden의 메모리 사용량이 확 줄어든 것을 볼 수 있었다. 

스크린샷에는 잡히지 않았지만 이 때 가득차있던 S0 영역이 완전히 비워지고 S1에 메모리 사용이 잡히기 시작했다. *S0, S1 어느 하나는 완전히 비어있어야 한다* 는 JVM 이론 시간의 공부가 눈 앞에서 확인되는 순간이었다. 

여러분도 집에서 심심할 때 한 번 돌려보시면 좋겠다. 물론 **공부하는 입장에서야 심심할 때 해볼 수 있는 일이겠지만 상용 서비스를 개발하는 개발자에게는 중요한 정보일 것이다.** 



