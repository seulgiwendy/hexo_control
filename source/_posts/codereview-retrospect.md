---
title: 지금까지 받아본 코드 리뷰를 정리해본다(1). 
date: 2019-3-25 08:40:00
categories:
    - java
    - codereview
---

## 지금까지 받아본 코드리뷰를 정리해본다. 

흔히 주니어 개발자가 성장할 수 있는 가장 좋은 방법은 코드리뷰를 충실히 받는 것이라고 이야기한다. 

성장 안하고 가만히 있어도 잘 먹고 잘 살 수 있는 직업을 갖게 되길 소망하지만(혹은 노동하지 않아도 잘 먹고 잘 살 수 있는 사회경제적 자본을 갖고 태어나기를 소망했지만) 당장 상황이 여의치 않으므로 일단 일을 더 잘 할 필요는 있다. 

성장하지 않고 가만히 있어도 되는 직업을 찾을 때까지는 힘 닿는대로 실력을 키울 필요가 있겠다. 

### 우리 조직의 코드리뷰 방향성

내가 몸담고 있는 네이버 쇼핑데이터플랫폼은 솔직히 말하면 그간 코드리뷰 문화가 그렇게 활성화되어 있던 조직은 아니다. 

업무에 비해 인력이 부족했고 특히 허리급 개발자가 부족한 것은 지금까지도 해결이 안 되고 있다. 

(우리 조직의 문은 [늘 열려있다.](https://recruit.navercorp.com/naver/job/detail/developer?annoId=20001611&classId=170&jobId=3809&entTypeCd=&searchTxt=%EC%87%BC%ED%95%91&searchSysComCd=))

최근 신입사원이 꽤 많이 충원되고 외부에서도 시니어급 개발자 분들이 몇몇 합류하셔서 상황이 조금씩 나아지고 있다. 특히 리더급 개발자들이 코드리뷰에 대한 강한 의지를 갖고있는 것은 꽤 고무적인 일이다(가끔 부담스러울 때도 있다).

개인적으로 감사함을 느끼는 선배님은 [이경일님](http://blog.leekyoungil.com) 으로 적극적으로 코드리뷰를 해주시는 것은 물론이고, 가끔은 리뷰를 해주시다 답답함을(!) 느끼시는지 직접 내 자리 옆으로 오셔서 즉석에서 페어 프로그래밍을 진행해주시기도 한다. 항상 꼰머로 비치지 않을까 조심하시지만 애초에 나와 년차가 10년 가까이 차이나는 시니어 개발자분이 그렇게 시간을 들여 코드리뷰를 해주시는 것 자체가 감사한 일이고, 또한 꼰머로 비칠까 걱정하며 조심해주시는 것은 감동적인 일이다. 

이경일님의 세심한 코드리뷰를 받고싶은 신입 개발자 준비생 분들은 다음 주제로 2019 네이버 캠퍼스 핵데이에 지원하시면 좋겠다.

Github issue: [쇼핑 실시간 상품 경매 플랫폼](https://github.com/NAVER-CAMPUS-HACKDAY/common/issues/6)

핵데이 원서접수 페이지 [바로가기](https://recruit.navercorp.com/naver/job/detail/developer?annoId=20002666&classId=&jobId=&entTypeCd=004&searchTxt=&searchSysComCd=)

### 내가 받은 코드리뷰 유형별 요약

코드리뷰를 받다보니 주로 **걸리는 쪽에서 계속 걸리는** 모습이 관측되었다. 

사람인 이상 어쩔 수 없는 부분이기도 하고, 앞으로 고쳐나가야 할 부분이기도 하다. 

가장 좋은 것은 안 고쳐도 잘 먹고살 수 있는 직업을 갖는 것이다. 하지만 당장 상황이 여의치 않으므로 일단은 고쳐나가면서 실력을 키울 필요가 있다. 

다음은 코드리뷰를 개략적으로, 유형별로 분류해 본 것이다. 

---

#### immutable해야 할 객체를 명확히 구분하자 

Java를 사용할 때 아무 생각없이 객체를 만들고 주고받지만, 내부의 element가 바뀌면 안 될 상황에서 immutability를 고려하지 않는 경우가 많다.

특히 java.util.Collection의 하위 클래스들을 사용하는 경우 이 점을 충분히 고려하여 불변해야 할 객체를 확정짓는 것이 좋다. 


```java
...
//아래의 shitUrls는 dumbSites 객체로부터 URL을 가져온다.
//언뜻 생각해보면 의도한 URL만 가져왔으므로 올바른 객체라고 생각하겠지만, 중간에 다른 String이 들어와도 아무도 모르게 된다. 

List<String> shitUrls = dumbSites.getUrls();

//Collections.unmodifiableList(Collections c); 을 사용하여 컬렉션 객체의 불변성을 확보한다. 

List<String> shitUrls = Collections.unmodifiableList(dumbSites.getUrls());
```

또한 parameter로 받을 객체, 지역 변수들에 대해서도, 최초 할당 이후 재할당이 필요없는 변수라면 final 예약어를 사용하여 최소한 객체 자체가 뒤바뀌는 일은 없도록 확실히 해 주자. 

```java
public String sayHelloToYou(String yourName) {
    //아래와 같이 지역 변수를 자유롭게 풀어주면 불변성을 확보하기 어렵게 된다. 
    String concatenatedName = yourName + "hi";
    
    //이와 같이 final을 붙여준다.
    final String concatenatedName = yourName + "hi";
}
```

당연한 얘기지만 **composed type**인 경우 setter를 통한 값의 변경은 피할 수 없을 것이다. 

그러나 그것은 객체 설계단의 문제이므로 여기서는 패스한다. 

---

#### 항상 null을 의식하라 

사실 이건 내가 Kotlin을 선호하는 이유이기도 하다. 

Kotlin이 null을 다루는 방식, 그리고 null-safety를 보장해주는 방식은 정말 우아하며 gorgeous 하고 phenomenal하다. 

그러나 당장 Kotlin을 사용하지 못하므로 Java에서 null을 안전하게 다룰 방법을 고민하고 연습하는 수밖에 없다. 


---
*Java에서 null의 그지같음(개인적 견해)*

null이 그지같은 이유는 NPE를 발생시키기 때문이다. 

사실 NPE만 아니라면, 혹은 NPE의 수퍼클래스가 RuntimeException이 아니라면 Java에서 null을 다루는 것이 그렇게 스트레스받는 일이 아닐 수도 있다. 

어차피 대부분의 상황에서 NPE는 에러로써 애플리케이션이 뻗어버릴 수밖에 없는데 차라리 try - catch로 잡아내는 것이 낫지 않겠는가? 개인적인 생각이다.

---

사실 **외부 API를 사용할 때** 랄지 **타인이 짜 놓은 코드에서 객체를 받아올 때** null 발생 가능성에 대한 의심을 별로 안한 채로 코드를 작성하게 되는 경우가 많다. 

그리고 종종 이는 대형 참사로 이어진다. 

코드리뷰에서는 사소한 null의 가능성도 주의깊게 살피라는 지적을 수 차례 받은 바 있다. 

```java
//iBATIS의 row handler 구현체를 작성한 코드다.

public void handleRow(Object o) {
    Shit shit = (Shit)o;
    
    //NPE가 발생할 포텐셜을 갖고 있는 코드다.
    shit.getToiletName();
}
```

상기한 사례의 경우, iBATIS는 Result Row가 존재하지 않으면 아예 Row Handler 객체를 호출하지 않으므로 null의 가능성은 없다(zero).

그러나 외부 API를 호출할 때, 결과물에 대한 확신이 없는 경우에는 늘 null에 대한 대비를 하는 것이 좋겠다. 

```java
...

@Autowired
private SewageConnector connector;

public void processShit() {
    Shit shit = connector.getFirstShit();

    //shit이 null 인 경우 진입하여 객체를 필요로 하는 로직을 건너뛰고 메소드를 끝내버린다.
    if(shit == null) {
        System.out.println("no shit available! clean!");
        return;
    }
    
    ...
}
```

어찌보면 참 사소한 것인데 실무에선 까먹기 십상이다.

---

#### try-with-resources 를 적극 활용하라

Closeable한 자원을 사용하는 경우 제때 닫아주지 않으면 성능에 악영향을 미친다. 

그동안 우리는 try - catch - finally 패턴을 많이 사용해왔다. 그러나 Java 1.7 이상만 되어도 훨씬 간결한 syntax의 try - with - resources를 활용할 수 있으므로 이를 적극 활용하자. 

```java
...

BufferedReader in = new BufferedReader(new FileReader(new File("~/Documents/resume")));

try {
    String line = in.readLine();
    System.out.println(line);
}
catch(Exception e) {
    e.printStackTrace()
}
finally {
    //예외 발생 여부에 상관없이 리더를 닫아야 하기 때문에 finally 절이 발생한다.
    in.close();
}
```

finally를 없애고 가독성 높은 코드를 만들기 위해 아래와 같이 리팩토링한다. 

```java
try(BufferedReader in = new BufferedReader(...)) {
    //여기에 로직이 담긴다.
} catch(Exception e) {
    e.printStackTrace();
}
```

---

#### 굳이 Spring Bean일 필요가 있는지 고려하라 

Spring Bean은 verbose한 싱글턴 패턴 코드 없이 싱글턴을 보장할 수 있는 좋은 방법이고, 

**특히 AOP, IoC, transaction propagation 등** 을 쉽게 구현할 수 있게 해 주는 좋은 도구이다. 

그러나 아래의 고려사항을 충분히 생각해볼 필요가 있다.

- 굳이 Singleton 상태일 이유가 있을까?
    - static method 사용하면 안되는가? 
    - Singleton 상태에서, 부가 기능이 추가될 이유가 있는가? 
- 모든 Spring Bean에서 이 객체를 주입받을 필요가 있는가?
    - scope를 고려하라
        - 만약 특정 환경(특히, Spring Batch에서)에서만 사용할 bean이 전체 scope로 구현될 경우, 불필요한 객체를 생산하는 비용만 초래될 뿐이다.
        - Bean의 갯수, 혹은 initializing logic이 Spring의 Application Context의 초기 기동 시간에 직접적인 영향을 끼친다는 점을 고려하라.
    - 실행 환경에 따라 사용되는 Bean이 다르진 않는가?
        - XML config의 경우 설정 파일을 분리하여 환경에 따른 bean instantiation이 가능하도록 설계하라. 
        - Java config, 특히 Spring Boot의 경우, @ActiveProfiles, @Profile을 적극적으로 활용하라. 

**DEV 프로필에서만 사용될 컴포넌트의 예시**

```java
@Service
@ActiveProfiles("dev")
public class RetardService {
    ...
}
```

**코드리뷰 받은 내용을 옮겨본다.**

*아래의 코드는 실무에서 사용한 코드가 아니다.*

**코드 리뷰 커멘트** 

> (...) spring component 보다는 static class 로 만드는게 어떨까 싶네요~ (spring bean 으로 만들 필요가 없는 class 인거 같다는 소리 ㅎㅎㅎ)


*리뷰 이전 코오드*

```java
//Component로 annotate 함으로써 모든 상황에서 객체화(instantiate)해야만 하는 클래스가 되어버렸다.
@Component
public class PoopService {

   //타 Spring Bean을 주입받거나, AOP적인 요소가 없으므로 굳이 Spring Bean으로 선언할 필요가 없는 클래스였다. 
    public void pumpPoop() {
        System.out.println("poop!");
    }
}
```

*개선*

```java
public class PoopService {
    
    //기본 생성자를 private으로 막아버림으로써, 객체화를 방지한다. 
    private PoopService() {

    }
    
    //static method이므로 객체를 만들지 않고 사용이 가능하다.
    public static void pumpPoop() {
        System.out.println("poop!");
    }
}
```

---

#### 랜덤 코드 생성에 UUID를 사용하라

처음 구현한 코드는 아래와 같았다.

```java
public class RandomGenerator {
    
    @Autowired
    private RandomCache cache;

    @Autowired
    private RandomStringGenerator stringGenerator;

    public String generateRandomCode() {
        String newString = stringGenerator.generate();

        //재귀 호출한다. base case는 cache.hasString(newString) == false
        return !cache.hasString(newString) ? newString : generateRandomCode();
    }
}
```

리뷰어께서 지적하신 문제점은 다음과 같다. 

- 재귀호출이 몇 번 일어날지 보장할 수 없다. 
    - 만약 cache에 1억건의 엔트리가 이미 들어있는 상황이라면?
    - 만약 RandomStringGenerator의 로직에 문제가 있어, 랜덤값이 보장이 안 되고 있는 상황이라면? 
- 문자열을 요청할때마다 내부적으로 일일이 비교연산이 일어날 텐데, 그 비용이 너무 커 보인다. 
- **UUID**를 활용하여 리팩토링한다.

*리팩토링 완료한 코드*
```java
public class RandomGenerator {
    
    ...

    public String generateRandomCode() {
        StringBuilder accessCode = new StringBuilder(UUID.randomUUID().toString());

        //후처리 로직을 삽입하고...
        ...

        return accessCode;
    }
}
```

---
### 1부 끝 

사실 앞으로도 정리해나갈 내용은 많을 것이다. 

하나의 포스팅으로 정리하기엔 내용이 방대한 것 같으니, 오늘은 여기서 글을 마쳐야겠다. 내가 특별히 지적 능력이 지진하거나 주니어 개발자로서의 소양이 부족한 것이 아니라면, 내가 받은 코드리뷰는 다른 주니어들도 마찬가지로 많이 받고 있거나, 혹은 받아야할 내용을 포함하고 있을 거라 본다. 

가장 좋은 것은 성장하지 않아도 먹고살 수 있는 직업을 갖는 것이겠지만, 일단은 그럴 수 없으므로 매일 매일 조금씩이라도 실력이 늘어야 할 것이다. 

#### 백기선님의 동영상 - 더 좋은 개발자가 되는 법

[!["나는 정말 이걸 좋아 하나?"](https://img.youtube.com/vi/yEcCQZlFmaM/0.jpg)](https://www.youtube.com/watch?v=yEcCQZlFmaM)

좋아하면 잘하게 된다. 좋아하기는 어려운 일인 것 같긴 하다. 

개발을 좋아해서 이 업을 시작했지만 취미/관심분야로써 즐긴 것과 직업인으로서 이 업을 즐기는 것은 전혀 다른 차원의 문제라는 것을 요즘 새삼 깨닫고 있다. 

나는 어디로 가야하나? 일단 열심히 달리는 수밖에 없을 것이다. 

#### "지금까지 받아본 코드 리뷰를 정리해본다." 시리즈는 앞으로 틈나는 대로 계속 정리할 생각이다. 
