---
title: TDD 흉내 내며 즐겁게 리팩토링하기. 
date: 2019-3-25 08:40:00
categories:
    - java
    - tdd
    - refactoring
---

## TDD를 하는 척 하며 즐겁게 리팩토링해보자. 

TDD처럼 논란의 한 복판에 서 있는 개발 방법론도 흔치 않을 것이다. 

> TDD 그거 완전 시간 낭비다. 하지마라. 번거롭다. *유닛 테스트도 커버리지에 집착할 필욘 없다.*

> TDD 좋기는 한데 실무 개발자들이 적용하기엔 분명한 한계가 있다. *그러나 테스트 커버리지는 높여나가자.*

> 나는 모든 개발자가 모든 실무 개발을 TDD base로 해야 한다고 생각한다. *지금 당장은 못하더라도 이상향을 거기로 잡자.*

모두 다 개발자로 취업한 후 들었던 말들이다. 

### TDD가 멀게 느껴지는 이유

학생때 [TDD를 너무나 좋아하는 나머지 삶의 방식 또한 TDD 기반으로 리팩토링 하고 계신 사부님](https://www.slideshare.net/OKJSP/okky-tdd) 께 Java를 배웠다. 그땐 실무 개발자들이 TDD를 하지 않는 것이 일종의 나태함의 징표는 아닐까 생각하기도 했었다. 정말 오만한 생각이었지만.... 

월급 받고 코드를 짠 지 1년 갓 넘긴 지금 시점, 실무 개발자들이 TDD를 하지 못하는 이유가 내 눈에도 명확히 보이기 시작했다. 

- 비즈니스 로직은 **하루에도 몇 번씩 바뀐다.** 
    - 학생 개발자 시절엔, 내가 (프로젝트를) 설계하고 내가 구현했으니 알 수 없는 어려움이었다.
    - 대다수의 조직과 기업에서 개발자는 비즈니스 로직에 대한 통제권을 갖지 못한다.

- 애플리케이션 자체가 **방대하다.**
    - 통합 테스트가 사실상 불가능하다. 
    - 테스트 환경을 완전히 독립시킬 수가 없다(데이터의 부재, 데이터소스의 병립 불가)

- **시간이 없다.**
- 팀 전체의 합의를 이루기가 어렵다.

### TDD 흉내를 내보기 위한 몸부림

업무 중 완전한 형태의 TDD까지 실천하진 못하더라도, 흉내라도 내보고 싶던 차에 그런 시도를 해볼 만한 개발과제가 잘 주어지지 않아 답답함을 느끼고 있었다. 

**그러다 기회가 왔다.**

#### 이 개발과제가 TDD 흉내내기에 적합했던 이유

상세한 내용은 업무상 기밀이라 밝히지 못하지만 과제는 아래와 같은 조건을 충족하고 있었다.

- 값 기반의 테스트가 가능하다. 
    - 로직의 멱등성이 보장된다. 
    - 임의의 값으로 테스트가 용이하다.
- 외부 종속성을 끊어낸 채로 모델 클래스만의 테스트와 개발이 가능하다. 
- 레거시 코드의 복잡도가 높지 않다.

*그래서 시작했는데....*

### 과제의 요구사항

과제의 요구사항은 다음과 같다. 

- DB에 담겨 있는 특정 문자열을 변환하라.
- 로그인한 사용자의 권한에 따라, 때로는 변환 전 문자열을 노출하고, 때로는 변환된 문자열을 노출해야 한다. 

### 작업 과정 

요구사항과 레거시 코드와 그 로직, 데이터베이스에 담겨있는 데이터의 개략적인 모습(?) 을 확인한 후 **테스트 코드 작성(요구사항 정의)** 을 시작하였다. 

```java

public class StringModificationTest {
    
    String originalString;
    BusinessModel model;

    @Before
    public void setUp() {
        this.originalString = //DB에 원래 담겨있어야 할 문자열
        this.model = new BusinessModel(this.originalString);
    }

    //비즈니스 로직 노출의 우려가 있어 메소드 이름이 모호한 것은 이해해주시길 바란다.
    @Test
    public void test_string_modified_correctly() {
        String expected = "expected_string";
        this.model.processString();

        assertThat(this.model.getString, is(expected));
    }
}
```

#### 문자열 변환 로직의 작성

문자열 변환 로직을 모델 클래스에 작성한다. 

```java
public class BusinessModel {
    ...

    public void processString() {
        String newString = this.modelString.....
        //문자열 처리 로직 작성

        this.modelString = newString;
    }
}
```

#### 다시 테스트 

테스트를 다시 돌린다.

![TDD phase](https://s3.amazonaws.com/mokacoding/2018-09-18-red-green-refactor.jpg)

#### 성공? 

로직 작성 -> 테스트 간에는 일단 **테스트의 성공** 에 집중한다. 테스트의 성공을 위해 수단과 방법을 가리지 않는다. *리팩토링/클린코드는 지금 단계에선 관심이 없다.*

극단적으로 보면 아래와 같은 구현을 하는 것도 의미가 없진 않다.

```java
public class BusinessModel {

    ...

    public void processString() {
        //테스트에서 기대했던 값을 그대로 넣어버리는 것이다. 
        this.modelString = "expected_string"
    }
}
```

이게 무슨 의미냐 하겠지만 나는 이 과정에서 몇 가지 의미를 본다. 

- test suite가 초록색으로 빛나는 것을 보는 거 자체가 성취감을 준다. 
    - 개발은 어차피 사람이 하는 것이기때문에 이런 성취감을 얻는 것 자체가 의미가 있다.
    - [작은 성공에 대한 영상](https://www.youtube.com/watch?v=OFN9mQVjPdU)
- **테스트 케이스에 대한 검증이 이뤄진다.** 
    - 모델 클래스의 수정으로 테스트가 성공할 수 있다는 보증을 받을 수 있다. 
    - 테스트 대상을 확실히하고 있음을 알 수 있다. 

#### 두 번째 성공을 향해

초록색으로 빛나는 test suite를 확인했다면 **이제는 일반화에 집중한다.** 

아까 첫 green light를 보기 위해 수단과 방법을 가리지 않았다면, 이제는 **최대한 많은 edge case에 대해 테스트가 성공하도록 로직을 정교화한다.**

```java
public void processString() {
        //당연히 다른 값에 대해서는 테스트가 실패할 것이다. 
        this.modelString = "expected_string"
    }
```

지금 단계에서부터는 **최대한 많은 값에 대한 테스트**를 확보하는 것이 중요하다.

테스트 코드를 수정한다. 

```java
@Test
    public void test_string_modified_correctly() {
        String expected = "expected_string";
        this.model.processString();

        assertThat(this.model.getString(), is(expected));

        String originalString2 = "original_string_2";
        String expected2 = "expected_string_2";

        this.model.setString(originalString2);
        this.model.processString();

        assertThat(this.model.getString(), is(expected2));
        
        ...
    }
```

사람이 하면 삽질이다. 값 기반 테스트를 도와주는 프레임워크의 적용을 검토해 본다.

아래 프레임워크가 유명해 보이지만 이번 작업에선 워낙 레거시 데이터베이스 내용들이 다사다난해서 적용해보지 못했다. 

- [jQwik](https://jqwik.net/)

#### 두 번째 성공 이후

API의 멱등성이 충분히 확보되었다고 느낀다면 **리팩토링을 통한 클린 코드의 구현에 집중한다.** 

우리는 이미 스펙에 대해 충분한 테스트 케이스를 갖고 있기 때문에 리팩토링의 사이드이펙트를 두려워할 이유가 없다. **테스트 케이스에 빨간 불이 들어온다면 다시 되돌아가거나 문제를 찾아 수정하면 그만이다**. 

비즈니스 로직 노출의 우려로.... 리팩토링 과정을 상세히 담지 못하는 점은 아쉽다. 

### 작업 회고

오랜만에 초록불이 짠 뜨는 경험을 하며 사무실에서 즐겁게 코딩을 했다. 굳이 유럽 여행중인 지금 기억을 되살려서 블로그 글을 쓰고싶을 정도로... 

그러나 결국 다음과 같은 문제는 예방하지 못했다. 

- 이미 데이터베이스에 입력되어 있는 값들의 edge case에 대해 충분히 검증하지 못했다. 
    - 기대하던 포맷에 어긋나는 데이터가 DB에 입력되어 있었고, 이에 따라 `ArrayIndexOutOfBoundsException` 이 발생함. 

어차피 애플리케이션 단에서 잡아낼 수 없는 문제였으므로 TDD 그 자체가 문제는 아니었다고 본다. 

그리고 소소한 문제로는...

- 작업 시간이 오래 걸리는 것은 어쩔 수 없다. 
    - 그러나 평소에 그만큼 막코딩을 하고 있었다는 반증일 수도 있다.
    - 평소보다 많이 오래걸리지는 않았다. 평소 작업 시간의 120% 정도가 소요되었다. 
- 개발 진척과정을 다른 팀원들과 활발히 공유하지 못했다. 

이와 같은 아쉬움을 경험했다. 

### 앞으로는? 

앞으로도 지금과 같은 시나리오에 적합한 개발과제가 주어진다면, 적극적으로 TDD를 흉내내며 개발해볼 생각이다. 

완전한 형태의 TDD를 행했는지는 잘 모르겠지만 흉내라도 내야 언젠가는 그 정점에 갈 수 있으리라 믿는다. 

또한 조금 더 활발한 커뮤니케이션으로 팀에 이런 개발문화를 전파시켜보려고 노력해보고 싶다. 물론 내 노력만으로는 이뤄지지 않을 일이란 것을 안다. 


