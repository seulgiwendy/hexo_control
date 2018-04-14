---
title: 아침 CS - (5) 해시 함수
date: 2018-04-13 07:59:34
categories:
    - morning_cs
    - hash
    - algorithm
---

### 해시 함수 

해시 함수는 프로그래밍을 하며 가장 흔하게 접할 수 있는 함수 중 하나로 자바의 `HashMap, HashSet` 등 자료형과 파이썬의 `Dictionary` 등에서 사용되고 있다. 

또 개발하며 `equals(), hashCode()`를 같이 오버라이드해본 경험은 다들 갖고 있을 것이다. 

왜 두 녀석이 따라다니는지, 해시코드가 뭔지, 그리고 해시 함수는 무슨 역할을 하는지 알아보자. 

---

#### 개념

**정의**

임의의 길이의 데이터를 **고정된 길이의 데이터**로 매핑하는 함수이다. 그리고 그 데이터를 해시 값, 해시 코드 혹은 해시 체크섬이라고도 부른다. 

![해시함수 개념도](https://upload.wikimedia.org/wikipedia/commons/thumb/5/58/Hash_table_4_1_1_0_0_1_0_LL.svg/300px-Hash_table_4_1_1_0_0_1_0_LL.svg.png)

사람의 이름을 해시코드로 바꿔주는 위 해시함수의 개념도에서 *고정된 길이* 는 두 자리의 정수일 것이다. 

위 개념도에서 나타난 해시 함수의 기본적인 성질이 두 가지가 있다. 

* 결정적: 해시값이 다르면 원래의 데이터도 다르다. 
* 단사 함수가 아님: 같은 해시값을 같더라도 원래의 입력값이 같은 것은 아니다. 
    * 위 그림을 보면 Sandra Dee, John Smith가 모두 같은 해시값 02를 갖지만, 원 데이터는 분명히 다르다. 

---

#### 해시의 충돌 

상기한 것처럼 다른 두 개 이상의 값이 같은 해시코드를 가질 수 있다. 

이러한 충돌은 알고리즘이나 자료구조의 효율성을 저해하기 때문에 최대한 억제해야 한다. 

**비둘기집의 원리**

해시코드가 왜 충돌하는지를 살펴보려면 먼저 비둘기집의 원리를 간단히 이해할 필요가 있겠다. 

> `n + 1`개의 물건을 `n`개의 상자에 넣은 경우, 최소한 한 상자에는 두 개 이상의 물건이 들어있다.


![비둘기집](https://upload.wikimedia.org/wikipedia/commons/5/5c/TooManyPigeons.jpg)


위 사진에서 만약 10마리의 비둘기를 보관하려고 했다면, 당연히 충돌이 발생할 것이다. 

해시 함수를 통해 처리되는 데이터의 범위보다 해시값의 범위가 더 좁다면 **당연히** 같은 원리로 충돌이 발생한다. 

---

#### 자바에서의 `hashCode()` 작동

사실 자바 자체가 메소드를 만들어주는 것이 아닌, 각자 사용하는 개발 환경(Eclipse, IntelliJ, 혹은 Google Commons 같은 외부 라이브러리....)에 따라 구현이 달라지기때문에 몇 가지의 대표적인 경우만 보고 넘어가도록 하겠다. 

**`java.lang.Object`** 에서의 존재

```java
public native int hashCode();
```

(...)  이게 끝이다. 

자바의 native 키워드에 대해서도 알면 좋겠지만 나중에 보도록 하고 일단 이 메소드가 어떻게 해시코드를 만들어주는지에 대해 알아보자. 

**작동 원리**

**OpenJDK** 에서의 구현 

네이티브 코드의 호출은 각 JVM 구현체별로 그 결과가 상이하게 나타날 수 있다. 

```cpp
JVM_ENTRY(jint, JVM_IHashCode(JNIEnv* env, jobject handle))
   JVMWrapper("JVM_IHashCode");
   // as implemented in the classic virtual machine; return 0 if object is NULL
      return handle == NULL ? 0 : ObjectSynchronizer::FastHashCode (THREAD, JNIHandles::resolve_non_null(handle)) ;
JVM_END
```

NULL 이라면 0을 돌려주고 뭔가 값이 있다면 `ObjectSynchronizer.FastHashCode()` 라는게 호출되는 것 같다. 

OpenJDK는 기본적으로 해시코드를 만드는 전략 여섯 개를 갖고 있는데, 다음과 같다. 

```cpp
  intptr_t value = 0 ;
  if (hashCode == 0) {
     // This form uses an unguarded global Park-Miller RNG,
     // so it's possible for two threads to race and generate the same RNG.
     // On MP system we'll have lots of RW access to a global, so the
     // mechanism induces lots of coherency traffic.
     value = os::random() ;
  } else
  if (hashCode == 1) {
     // This variation has the property of being stable (idempotent)
     // between STW operations.  This can be useful in some of the 1-0
     // synchronization schemes.
     intptr_t addrBits = cast_from_oop<intptr_t>(obj) >> 3 ;
     value = addrBits ^ (addrBits >> 5) ^ GVars.stwRandom ;
  } else
  if (hashCode == 2) {
     value = 1 ;            // for sensitivity testing
  } else
  if (hashCode == 3) {
     value = ++GVars.hcSequence ;
  } else
  if (hashCode == 4) {
     value = cast_from_oop<intptr_t>(obj) ;
  } else {
     // Marsaglia's xor-shift scheme with thread-specific state
     // This is probably the best overall implementation -- we'll
     // likely make this the default in future releases.
     unsigned t = Self->_hashStateX ;
     t ^= (t << 11) ;
     Self->_hashStateX = Self->_hashStateY ;
     Self->_hashStateY = Self->_hashStateZ ;
     Self->_hashStateZ = Self->_hashStateW ;
     unsigned v = Self->_hashStateW ;
     v = (v ^ (v >> 19)) ^ (t ^ (t >> 8)) ;
     Self->_hashStateW = v ;
     value = v ;
  }
  ```

정리해보면.... 
  
| 번호        | 작동방식           | 비고
| ------------- |:-------------:| -----:|
| 0     | 완전 랜덤 숫자 출력 | 아예 랜덤 숫자임. Park-Miller 난수 생성 방식 [참고](https://en.wikipedia.org/wiki/Lehmer_random_number_generator) |
| 1      | 메모리 주소 함수      |   - |
| 2 | 하드코딩된 1      |    민감성 체크 용 |
| 3 | 시퀸스 | - |
| 4 | 메모리 주소 | - |
| 5 | XOR shift 난수생성법과 혼합된 스레드 상태 | [참고](https://en.wikipedia.org/wiki/Xorshift)

솔직히 어떤 상황에서 어떤 전략이 사용될지 보장할 수도 없고(기본값이 4번이기는 하다, 우리가 흔히 알고있는 것처럼) 저 다섯가지를 바꿔가며 쓸 필요가 있는 것도 (일상적인 상황에서는) 아니다. 

**그냥 각자 IDE가 만들어주는 해시함수 잘 쓰자.**

**참고 문헌**

[OpenJDK source](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/87ee5ee27509/src/share/vm/runtime/synchronizer.cpp)

[How does the default hashCode() work?](https://srvaroa.github.io/jvm/java/openjdk/biased-locking/2017/01/30/hashCode.html)