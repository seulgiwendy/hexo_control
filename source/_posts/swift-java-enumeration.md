---
title: Swift 맛보기(1) - Java와 Swift의 열거형을 비교해보다. 
date: 2019-6-10 08:40:00
categories:
    - java
    - swift
---

## Swift로 조금씩 뭔가 만들어보고 있다. 

무료하고 때론 우울하기까지 한 개발자 생활에 약간의 변화가 필요한 시점인 것 같아, 완전히 다른 언어를 배우고 있다. 

JVM 바깥 세상으로 길을 나서는 것이 두렵게 느껴지기까지 했지만 새로 학습할 언어로 [Swift](https://swift.org/) 를 선택하였다. 

개발 공부를 시작한 이래로 줄곧 서버측 개발에 즐겨 사용되는 언어와 프레임워크만을 학습하였다. Python으로 처음 개발에 입문했고, 이후 Java, Ruby, Scala, Kotlin(이 언어들을 다 잘한다는 건 아니다)을 학습하며 주로 서버 구축, 서버 사이드 앱 개발에만 집중해왔다. 

나의 경쟁력(?) 을 늘리는 측면도 있지만 앞으로 1인 개발자로서 나만의 비즈니스를 하게될 때, 혹은 정말 재미로 이것저것 만들어보게 될 때 클라이언트 사이드 개발이 꼭 필요하게 될 것 같아 큰 맘 먹고 Swift와 Swift를 이용한 iOS 앱 프로그래밍까지 학습해보려고 한다. *서버사이드 개발자들보다 클라이언트 개발자들이 더 재밌게 사는 것 같기도 하다.*

학습 방법은 내가 기존에 경험했던 그대로, 두꺼운 책을 보며 시작하지 않고 일단 뭐라도 만들어보며 배우려고 한다. 코드스쿼드 시절 부여받았던 다양한 도전과제들을 Swift로 구현해보는 것이 좋은 출발이 될 것이다. 

## 그래서 로또게임을 구현해보고 있다. 

내가 생각하기에 간단한 콘솔 기반 로또게임은 새로운 언어를 배우기에 정말 좋은 프로젝트다. 우선, 적당한 수준의 분기처리가 존재하고, 적당한 수준의 stdin-stdout 함수를 활용할 수 있으며, 적당한 수준의 문자열 포맷팅을 경험할 수 있다. 

맘먹기에 따라 날코딩으로 700줄짜리 통짜 클래스로 구현할 수도 있고 객체지향의 묘를 살려 관심사의 분리, 역할의 위임, 느슨한 결합 등을 구현해보는 것도 얼마든지 가능하다(자바로 이 짓을 하다가 결국 포비로부터 머지받지 못하였다). 

오늘 살펴볼 부분은 로또게임의 당첨 등수를 열거형(enum)으로 표현하며 느낀, Java와 Swift 사이의 열거형의 차이점이다. 

### enum은 간단한 개념이다. 

말 그대로 열거해주면 된다. "열거" 라는 표현법 그 자체는 두 언어가 크게 다르지 않다. 

```java
public enum PortalSites {
    NAVER,
    GODKAO,
    GODGLE
}
```

```swift
enum PortalSites {
    case naver, godkao, godgle
}
```

case라는 키워드가 앞에 붙는다는 점 빼곤 크게 다를 바가 없어 보인다. 

**enum의 호출**

변수에 할당하는 방법도 양 언어가 거의 같다. 

```java
PortalSites naver = PortalSites.NAVER;
```

```swift
var naver = PortalSites.naver
```

**enum에 행위 추가하기**

차이점은 여기서 발생하는 것 같다. 

열거형에 행위를 추가하려면 Swift의 경우 `protocol`, Java의 경우 `interface`를 구현시켜야 하는 점은 비슷하다. 

아까 구현한 `PortalSites` enum에 홈페이지 주소를 리턴하는 행위를 추가하는 상황을 가정해 보자. 

```java
public interface PortalSiteAction {
    String getHomepageUrl();
}
```

```swift
protocol PortalSiteHomepage {
    var homepageUrl: String {get}
}
```

여기까진 비슷한 것 같다. 사실 여기까지만 보고 Swift에서의 구현도 Java와 별 다를게 없다고 생각했었다. 

그러나 [다른 분들의 코드](https://github.com/code-squad/swift-cardgame/blob/youth27/CardGame/CardGame/CardDeck.swift)를 참고하다보니 아래와 같이 `switch` 문으로 행위를 분기처리하는것이 인상적이었다. 

```swift
enum Denomination: Int, CustomStringConvertible, EnumCollection, Comparable {
        case two = 2, three, four, five, six, seven, eight, nine, ten, eleven, twelve, thirteen, ace
        
        var description: String {
            switch self {
            case .ace: return "A"
            case .eleven: return "J"
            case .twelve: return "Q"
            case .thirteen: return "K"
            default: return String(self.rawValue)
            }
        }
    (...)
```

상기한 사례를 참고해 `PortalSites` enum을 완성시키면 아래와 같이 될 것이다. 

```swift
enum PortalSites: PortalSiteHomepage {
    case naver, godkao, godgle

    var homepageUrl: String {
        switch self {
            case .naver: return "https://www.naver.com"
            case .godkao: return "https://www.daum.net"
            case .godgle: return "https://www.google.com"
        }
    }
}
```
인상적인 문법이라고 생각됐다. 

**인상적으로 느껴진 이유** 

- Java의 `interface`에 상응하는 개념인 `protocol`이 메소드와 리턴 타입에 집중하지 않고, 변수 그 자체를 공통으로 선언할 수 있게 해준 점이 인상적이었다. 

- 각각의 enum이 가져야 할 공통 변수를 선언하고, `switch` 문을 적극적으로 사용하여 코드량을 줄인 것이 인상적이었다. 

- 같은 분량의 확장을 할 때에, Java보다 작성해야 할 코드가 훨씬 줄어들 것 같다. 

**Java에 적용해볼 수는 없을까?**

Java의 enum도 인터페이스를 구현할 때에, 공통 메소드로서 구현하는 것이 가능하기 때문에, 아래와 같은 구현도 가능해진다. 

```java
public enum PortalSites implements PortalSiteAction {
    NAVER,
    GODKAO,
    GODGLE

    public String getHomepageUrl() {
        switch(this) {
            case NAVER:
                return "https://www.naver.com";
            case GODKAO:
                return "https://www.daum.net";
            case GODGLE:
                return "https://www.google.com";
        }
    }
}
```

사실 이런 식의 구현은 지금까지 코딩하며 시도해본 적도 없고 이렇게 구현한 사례를 본 기억도 잘 없다. 

그러나 enum을 추가할 때마다 로직이 바뀌어야만 하는 부분이 있다면, 이런 종류의 분기처리를 시도해보는 것도 괜찮을 것 같다. 

특히 functional interface와 결합한다면 클린 코드를 만드는 데에 더 큰 도움이 될 것 같다. 
