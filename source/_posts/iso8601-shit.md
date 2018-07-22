---
title: ISO 8601 대 삽질기
date: 2018-07-12 16:42:58
categories:
    - java
    - datetimeformatter
    - localdatetime
---

### ISO 8601 대 삽질기

최근 게시물을 관리하는 간단한 코드를 구현하며 시간 정보를 관리할 일이 있었다. 

데이터 원본 상의 column type은 DATE(ORACLE)이었지만 iBATIS를 사용하다보니 매핑이 여의치 않아 그냥 String type으로 데이터를 갖고 오다보니 재밌고 빡치는 해프닝을 겪어, 관련한 사항을 잊지 않기 위해 여기에 이렇게 기록해본다. 

#### 문제의 발단 

해당 엔티티는 아래와 같은 코드를 통해 객체가 생성된 현재시점의 시간정보를 생성하고 있다. *(실제 작성중인 코드와는 상이함을 미리 알려드린다.)*

```java
private static String currentTime() {
    return DateTimeFormatter.ofPattern("yyyy-mm-dd").format(LocalDateTime.now());
}
```

그러자 아래와 같은 우스꽝스러운 시간이 표시되기 시작했다. 

```
2018-48-12
2018-51-12
```

#### 왜 그럴까?

ISO 8601 표기법상의 시간 / 일자 코드를 제대로 숙지하지 못해서 벌어진 일이었다.

**소문자 m과 대문자 M의 차이를 바로 알고 사용해야겠다.**

m(lower case) : minute of hour
M(upper case) : month of year

*헷갈리지 말자.*
