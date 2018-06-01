---
title: java.lang.String을 파 보았다
date: 2018-05-21 09:15:39
categories:
    - Java
    - String
---

### 자바의 문자열에 대해 제대로 공부해보자. 

아마 어떤 언어를 배우든 그 시작은 "문자열 다루기" 일 것이다. "다룬다" 는 표현이 거창한가? 그렇지만 프로그래밍의 시작을 하려거든 어떤 식으로든 문자열과 만나는 경우가 많다.

```python
def helloWorld():
    print("hello world!")
```

쌍따옴표 안에 감싸진 것은 분명히 **문자열이다.** 

자바는 뭐가 다를까? 

```java
public static void main (String[] args) {
    System.out.println("hello world!");
}
```

문자열 없이는 첫 발짝도 **뗼 수 없다.** 우리는 프로그래밍을 학습할 때 문자열을 가장 먼저 배운다. 

어쩌면 문자열은 프로그래밍 언어에서 등장하는 유일한 "인간의 언어" 일지도 모르겠다는 생각이 든다. 

---

#### 오해

문자열, 특히 Java에서의 문자열은 처음 배우는 개념인만큼 오개념이 생기기도 쉽고, 오개념이 잡혀도 오개념인 줄도 모르기때문에 나중에 바로잡기도 쉽지 않다. 

뭐 완전 뉴비 시절에는 이런 실수도 종종 하긴 한다. 

```java
boolean b = "안녕하세요" == "안녕하세요";
```

문자열을 ==으로 비교한다던지,

```java
String hi = new String("Hellow");
```

문자열에 생성자를 사용한다던지 하는 실수들 말이다. 

이런 실수야 각자 해결하고 이 자리까지 왔을테니 굳이 언급하진 않겠다. 이번 글에서는 Java의 j 자 정도에 대한 이해가 생긴 프로그래머들조차도 흔히 갖는 오해, 그리고 흔히 하는 실수들에 대해 생각해보고자 한다. 

---
**java.lang.String은 어떻게 생성자 없이도 객체가 생성되나?**

궁금증 하나로 출발해보고 싶다. 우리는 Java에서 문자열은 *객체* 라고 배웠다. 

그러나 생성자는 쓰지 말라고 한다. 어디서 이런 미스매치가 발생한 걸까? 

```java
String a = "hello"
```

라는 짤막한 문자열 선언이, 컴파일러를 통해 아래와 같이 바뀐다.

```java
String a = new String(new char[]{'h', 'e', 'l', 'l', 'o'}).intern();
```

.intern() 이 뭐냐면.... 일단 java.lang.String 에는 이렇게 돼 있다. 

```java
public native String intern();
```

native 코드를 호출하는 것 같다. 

이 짤막한 메소드의 기능은 문자열 풀링(pooling)을 담당하는 것이다. 즉, 우리가 문자열을 만들면 우리는 우리가 만든 객체를 그대로 사용하는 게 아닌, JVM 내부에 형성된 literal pool의 객체를 가져다 쓰게 된다. 

처음 문자열이 선언됐을 때 .intern()이 호출되고, 이 메소드는 JVM의 문자열 풀에 해당 객체와 같은 내용의 문자열이 기 존재하는지를 질의한다. 이때 .equals()를 사용한다. 

만약 이미 객체가 있다면 문자열 풀은 해당 객체를 리턴하고, **없다면 만들어진 문자열 객체를 문자열 풀에 집어넣은 후, 그 객체로 가는 참조를 리턴해준다.** 즉 어떠한 경우에도 우리가 문자열 객체의 생명주기를 직접 통제할 수는 없다. 

그래서 이런 결과가 발생하는 것이다. 

```java
System.out.println("a" == "a");
//true가 출력된다.
```

그러나 모든 문자열 생성이 다 .intern()을 타는 것은 아니다. **동적으로 문자열을 만들어주는 대부분의 메소드에서 만들어진 문자열은 new String() 을 쓴다.** .intern() 안 탄다. 

위와 같이 비교할 수 있다고 해서 .equals()를 안 써도 되는 게 아니란 얘기다. 

말장난 아니냐고 하면 할 말은 없는데 어쨌든 **객체가 풀링되고 있다는 사실이 중요한거다.** 비슷한 원리를 따르는 JVM의 신비가 또 하나 있는데 바로 wrapper class interning이다. 

아래 코드는 true일까, false일까? 

```java
Integer i1 = 1;
Integer i2 = 1;

i1 == i2;

```

이거 true. **객체는 equals로 비교하라며?**

자, 그럼 아래 코드는 true일까 false일까? 

```java
Integer big1 = 1024;
Integer big2 = 1024;

big1 == big2

```

이거 false다. *ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ*

**왜 그럴까?**

퍼포먼스 튜닝 목적으로 Integer라는 wrapper class에 대해 -127 ~ 128 사이의 값을 **미리 보관해서 별도의 배열에 보관하고 있기 때문이다.**

java.lang.Integer를 까보면 아래와 같은 구현이 나온다. 

```java
private static class IntegerCache {
         private IntegerCache(){}
 
         static final Integer cache[] = new Integer[-(-128) + 127 + 1];
 
         static {
             for(int i = 0; i < cache.length; i++)
                 cache[i] = new Integer(i - 128);
         }
     }
```

Integer 클래스가 최초로 사용됐을 때 무조건 한 번 돌게 돼 있다. 

Integer도 문자열과 같은 원리의 interning을 수행하고 있는 모습이다. 

---
**문자열은 객체인데 어떻게 + 연산자가 먹나?**

String 객체 어딘가에 해당 연산을 수행해주는 메소드가 있고, 그 메소드를 호출하는 것으로 컴파일 타임에 바꿔치기되기 때문은 아닐까?

```java
String s = "a" + "b"
```

의 연산은, 

```java
String s = new StringBuilder("s").append("b").toString();
```

으로 실행된다. 

*어, 그러면 StringBuilder 안 써도 되겠네요?*

**아뇨.** 

개발자의 판단과 책임으로, 문자열 더하기 연산이 지나치게 많이 반복되는 경우 처음부터 깔끔하게 StringBuilder를 사용하는 것이 좋다. 

```java
String hello = "hello";

for(int i = 0; i < 10; i++) {
    hello += "hi";
}
```

당신은 조금 전 10개의 스트링 빌더 객체를 생성해, 고작 "hi" 두 글자 더하고 내다 버렸다. 

동양 컴퓨터공학에서는 이렇게 쉽게 객체를 버리는 경우 컴퓨터의 균형이 깨진다고 본다. 

```java
StringBuilder hello = new StringBuilder("hello");

for(String hi: hiList) {
    hello.append(hi);
}

return hello.toString();
```

이게 훨씬 좋은 구현이다. 

**추가**

정상혁([Sanghyuck Jung](https://www.facebook.com/benelog))님이 아래와 같은 내용을 보충해 주셨다. 

*Java 9에서의 String concat 최적화* - StringConcatFactory 로 연결된다. (JDK 9 이상에서만 존재)

[http://openjdk.java.net/jeps/280](http://openjdk.java.net/jeps/280)



**.concat()이란 것도 있다면서요?**

있다.

```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    char buf[] = new char[count + otherLen];
    getChars(0, count, buf, 0);
    str.getChars(0, otherLen, buf, count);
    return new String(0, count + otherLen, buf);
}
```

이렇게 생겼다. 그러나 **일부러 이걸 쓸 일은 없을 것이다.**

보면 알겠지만 합칠 문자열을 받고, 그 문자열의 길이를 파악한 후, 그 문자열 길이 + 원 문자열 길이 만큼의 배열을 선언하고, 거기다가 원래 문자열의 CharArray를 덮고, 거기다가 합칠 문자열으 내용을 덮고, 그걸 또 새로운 String 객체의 생성자 파라메터로 넘겨서 객체를 만들어서 돌려주는 것이다. 

이렇게 만들어지는 객체는 오버헤드를 엄청나게 유발할 뿐만 아니라 앞서 설명한 interning도 안 되어있는 것이다. 

위의 내용 중에 오타가 있는 건 나도 알지만 그냥 쓰다보니 빡쳐서 바로잡지 않은 것이다. 

**쓰지 마세요.**

---

*StringBuilder도 있고, StringBuffer도 있잖아요?*

맞다. **두 클래스의 차이에 대해 인지한 후 정확한 상황판단 하에 사용해라.**

단순 취향 차이가 아닌 실제 퍼포먼스에서의 차이가 상당 부분 발생한다. 

코드를 보자. 


```java
abstract  class AbstractStringBuilder implements Appendable,CharSequence {

char[] value;

int count;

AbstractStringBuilder() {}

AbstractStringBuilder(int capacity) {

          value = new char[capacity];

    }
```

StringBuilder의 super class인 AbstractStringBuilder인데 char[] 로 문자열에 대한 관리를 하고 있음을 알 수 있다. 

그럼 StringBuffer는? AbstractStringBuilder를 extend하는 것은 같지만....

```java
private transient char[] toStringCache;
```

문제의 코드다. char[] 를 transient로 관리하고 있는 모습이다. 

뿐만 아니다. 

```java
@Override
public synchronized char More ...charAt(int index) {
    if ((index < 0) || (index >= count))
        throw new StringIndexOutOfBoundsException(index);
    return value[index];
}
```

```java
@Override
public synchronized StringBuffer More ...append(CharSequence s) {
    toStringCache = null;
    super.append(s);
    return this;
}
```

모든 메소드에 synchronized 예약어가 달려있다. 느려질 수밖에 없는 구조다. 

[synchronized가 왜 느릴까요?](https://stackoverflow.com/questions/1671089/why-are-synchronize-expensive-in-java)

JavaDoc에서 충분히 밝히고 있는 것처럼 StringBuffer는 thread-safe함이 요구되는 상황에서만 **제한적으로** 사용하는 것이 맞다. 남발할 경우 실행 속도가 느려질 수밖에 없을 것이다. 

개인적인 실험을 해본 적이 있었는데 StringBuilder와 StringBuffer의 .append()를 1억번 정도 콜한 경우 대략 3초 내외의 퍼포먼스 차이가 났던 것으로 기억한다. 

3초면 3000ms. 꽤 큰 차이다.

---

#### 마무리

사실 우리 모두가 프로그래밍 좀 하다보면 문자열에 대한 이해가 출중할 거라 자부하지만 의외로 오개념을 갖고있을 수도 있다. 

나도 이번 포스팅을 준비하며 찾아본 자료들에서 신선한 충격을 받은 경험이 꽤 있었다.

어쨌든, 기초가 참 중요한 것 같다. 익숙한 것부터 바로, 잘 알자!





