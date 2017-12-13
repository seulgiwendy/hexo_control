---
title: Getter에 대한 사소한 궁금증 - Getter는 Optional을 사용하면 안 될까?
date: 2017-12-03 17:53:02
tags:
    - Java
    - Functional_programming
    - Optional
---

### Java의 Getter는 Optional을 돌려주면 안 될까? 

```
"Null은 나의 백만 불 짜리 실수다."
-찰스 앤서니 리처드 호어, 영국의 컴퓨터 과학자, Null의 창시자
```

개발 공부를 시작하던 몇 달 전에 가장 이해하기 어려운 개념은 재귀함수도 아니고 인터페이스도 아닌 Null 값이었다. 

Null의 모호성 때문이었다. 이 값(?)은 값이 없다고 하기에도 모호하고 그렇다고 뭔가를 나타내고 있는 것도 아니다. return 값으로도 사용할 수 있고 인스턴스 변수 자리에도 잘 들어갈 수 있으며 그 당시에는 뭔가 문자열같다(!) 는 느낌도 받았었다. 그래서 아래와 같은 괴상한 코드를 짠 일도 있다. 

```
String foo = null;
if (foo == "null") {
    System.out.println("foo is null!");
}
```
지금 보면 헛웃음밖에 나오지 않는 정말 우스꽝스러운 코드 뭉치이지만 **그땐 진지했다.** 객체 개념, 클래스 개념이 확실하게 서 있지 않던 시기에 짠 코드들은 툭하면 NPE를 뱉어 댔고 Null값을 어떻게 처리할 지도 난감했었던 기억이 난다. 

개발 공부를 시작한 뒤 시간이 좀 흘러, 적어도 *Null이 문자열은 아니다* 라는 사실을 알게된 시점부터 나는 아래와 같은 Null check 로직 (내 맘대로 갖다 붙인 이름이다)을 구현하기 시작했다. 

```
public class Foo {
    
    private String bar;
    
    public String getBar() {
        if (bar == null) {
            return "Fuck You! There is no Bar anymore!";
        }
        return this.bar;
    }
}
```

이런 식의 구현은 한동안 계속되었던 것으로 기억한다. 아니, 솔직히 말하면 **이 이상의 방어적이고 효율적인 구현은 있을 수 없다**고 생각했었다. 나는 공부를 시작하던 시점부터 Java 8 환경에서 작업했으므로 레거시 이슈 같은 것은 전혀 아니었고....내가 Java 8에 대한 충분한 이해 없이 개발 공부를 시작한 것에 따른 부족함이었다. 

공부에 점점 깊이를 더해가게 되면서 함수형 프로그래밍의 개념과 Java 8의 Stream, Lambda와 같은 학습 주제들을 공부하게 되고, Optional에 대해 알게되면서 Null값을 아래와 같이 wrapping하는 일이 익숙해지게 되었다. 

```
Optional<String> foo = Bar.getFoo();
String fuck = foo.orElseThrow(() -> new NoSuchElementException("no value!"));
```

그런데 위의 코드를 보면 Bar 클래스의 Getter인 .getFoo()가 Optional을 돌려주고 있는 것을 알 수 있고, Getter가 Optional을 돌려주는 위와 같은 상황을 *지금까지의 개발 학습과 코드 리뷰, 깃허브 레파지토리 서핑에서 별로 본 적이 없다.*

심지어 lombok도 Optional을 돌려주는 Getter를 만들어주는 옵션이 없다. **Getter 메소드는 Optional을 돌려주면 안 되는 걸까?** 문득 든 궁금증이지만 한 두 시간 동안 구글을 뒤져 가며 내린 나름의 결론을 기록해 두고자 한다. 

(나름의 결론이니까 틀릴 수 있고 아마 틀릴 거다. 그래도 기록이란 게 그 자체로 의미가 있으니 선배 개발자들과 공유하며 다양한 의견 들어보고 싶다.)



#### 왜 Optional을 리턴하는 Getter는 없냐. 

Stack Overflow의 [Should Java 8 getters return Optional type?](https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type) 라는 글을 함께 읽어보며 논의를 전개해 나가고 싶다.

```
"...Optional type introduced in Java 8 is a new thing for many developers.

Is a getter method returning Optional<Foo> type in place of the classic Foo a good practice? Assume that the value can be null."

-자바 8의 옵셔널은 많은 개발자들에게 새로운 기능입니다(주: 이 글은 3년 전에 작성되었다). Foo라는 클래식 타입 대신 Optional로 감싼 값을 돌려주는 Getter를 만드는 것이 좋은 구현인가요? 값이 null일 수 있다는 전제 하에..."
```

**나와 똑같은 고민을 (3년 전에) 누가 이미 한 모습이다.**

가장 많은 추천을 받은 답변을 아래에 조금씩 옮겨보겠다. 

```
"...Of course, people will do what they want. But we did have a clear intention when adding this feature, and it was not to be a general purpose Maybe or Some type, as much as many people would have liked us to do so. Our intention was to provide a limited mechanism for library method return types where there needed to be a clear way to represent "no result", and using null for such was overwhelmingly likely to cause errors."

"...For example, you probably should never use it for something that returns an array of results, or a list of results; instead return an empty array or list. You should almost never use it as a field of something or a method parameter.

I think routinely using it as a return value for getters would definitely be over-use."

"...It is important to note that the intention of the Optional class is not to replace every single null reference. Instead, its purpose is to help design more-comprehensible APIs so that by just reading the signature of a method, you can tell whether you can expect an optional value. This forces you to actively unwrap an Optional to deal with the absence of a value."
```

**미국말이지만 이야기하고자 하는 바는 명쾌하다.** Optional을 맹신하거나 오남용하지 말라! 필요한 곳에만 사용하라! *값이 없을 상황이 명백하게 기대되는 경우에만 사용하라!* 정도로 요약해 볼 수 있을 것 같다. 

#### 그렇다면 Optional을 돌려주는 Getter는 의미있는가?

나의 생각으로 내린 결론부터 말하자면, **95%의 경우에 의미가 없을 거라고 본다.** 

첫째, 값을 꺼내오기 위해 만든 Getter가 **찾아올 값이 없을 거라고 기대하는 상황 자체가 자기 모순이다.** 우리가 Getter 메소드를 구현하는 것은 인스턴스가 의미 있는 값을 가지고 있고, 그 값을 직접 변형하거나 사용하기 위해 그렇게 하는 것이다. 

만약 Getter 메소드가 찾아올 값이 null일 수도 있다면 **애초에 그 메소드는 Getter가 아니어야 타당하다.** 즉, null을 돌려줄 수 있는 Getter가 설계된 상황은 그 자체로 충분한 객체지향적 설계를 하지 않은 상황이다. 

만약 Getter가 null을 다뤄야한다면 우리는 그 값을 직접 꺼내오는 메소드가 아닌 그 값을 인스턴스 차원에서 변형하고, 바꿀 수 있는 메소드를 구현해야 한다. 

Optional의 도입 취지를 다시 한 번 기억하자. Optional은 *의미 있는 null을 위한 보금자리* 라고 내게는 이해된다. Getter가 null을 돌려준다면, 그건 의미 있는 null일까? 다시 곰곰히 생각해보면 분명히 아닐 거라 생각된다. 

둘째, 레거시 프레임웍들과의 호환성 문제가 있다. Java Bean의 표준 규약으로서의 Getter는 Optional을 돌려주지 않기 때문에, Getter를 통해 객체에서 값을 받아다 써야 하는 다양한 프레임워크들과의 호환성 문제가 분명히 생길 것이다.

마지막으로, 불필요한 코딩량의 증가가 우려된다. null값이 별로 기대되지도 않는 맥락 속에서 Optional로 wrapping된 값을 가져오기 위해 추가적인 코드 작성이 필요해지는 상황이 연출되는 것이다. 

아래 예제를 통해 살펴보고 싶다. 

```
public class Bar {
    private String what;

    ...
}

String foo = Bar.getWhat(); 
```

```
public class Foo {
    private String what; 

    private Optional<String> getWhat() {
        return Optional.ofNullable(this.what);
    }
}

String foo = Foo.getWhat().orElseThrow(() -> new NoSuchElementException("no value!"));
```

의미있는 null의 사용이 기대되지 않는 상황에서, null 값을 굳이 Optional로 감싸고, 그렇게 감싸진 값을 꺼내오기 위해 추가적인 코딩 분량이 발생하는 상황이 그렇게 좋아보이지 않는다. 

Optional 객체에서 .get()을 호출하여 바로 값을 꺼내오면 되지 않나? **그런 구현은 정말 지양해야 한다고 본다.** .get()을 부르는 것은 Optional을 Optional답지 않게 사용하고 있는 모습이라 생각된다. .get()은 null값이 발생할 가능성을 일축하는 메소드라고 볼 수 있는데, 그러면 애초에 왜 리턴값으로 Optional을 사용하기로 결정했나? 생각해 볼 필요도 없는 문제라고 본다. 

### 결론: Getter의 리턴 값으로 Optional 객체를 사용하는 것은 의미 없다. 

Getter의 리턴 값으로 Optional을 돌려주는 것은 *의미도 없고, 호환성 문제가 발생할 수 있으며, 불필요한 코딩 분량까지 발생시킬 여지가 있다.* 

Getter의 구현을 최소화하는 것이 객체지향젹인 사고방식이라는 말을 들은 적이 있다. 그 말씀에 전적으로 동의한다. 값을 꺼내오지 말고 값의 처리결과를 받아보는 것이 최선의 선택이다. 

다시 한 번 강조하고 싶은데, 내가 생각하기에 null을 돌려줄 수도 있는 Getter 메소드는 정말 필요한 Getter인지 다시 한 번 클래스의 설계를 점검해보는 것이 좋겠다. Getter가 의미 있는 null을 반환해야 하는 상황은 정말 겪기 어렵다. 

#### 추가 학습 과제 

* 정말 Optional을 돌려주는 Getter가 내 주변에 없는지 다시 한 번 찾아본다. 
* lombok이 Getter를 만들어주는 원리에 대해 학습한다. 
* Java Bean의 표준 규약에 대해 다시 한 번 학습한다. 



