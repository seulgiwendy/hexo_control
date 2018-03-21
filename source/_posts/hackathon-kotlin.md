---
title: 해커톤 나가서 삽질하기 (1) - 코틀린으로 DTO를 만들어보자. 
date: 2018-03-19 11:59:48
tags:
    - hackathon
    - kotlin
    - java
    - Spring
    
---

### 해커톤 나가서 삽질하기 (1) - Kotlin으로 DTO를 만들어보자. 

템플릿 엔진을 사용하지 않는 개발, 즉 프론트 프로젝트와 백엔드 프로젝트가 분리되어 있는 프로젝트의 가장 큰 난관은 **데이터셋을 획정** 하는 일이다. 

누구나 시작할 땐 그럴싸한 계획이 있다. Git repo에 Wiki를 운영하자, PR을 보내기 전 미리 협의하자, RESTful하게 하자..... 그러나 이런 계획은 아마추어들의 개발 문화 하에서는 거의 지켜지지 않는다. 평소에 개발해도 지켜지지 않는데 **24시간, 혹은 48시간 같은 촉박한 시간에 개발해야하는 해커톤** 에서는 말할 필요도 없을 것이다. 

> 그러니까 백엔드 개발자는, DTO를 작성할 때 이게 반드시 바뀔 것이란(...) 확신을 가지고 언제든 수정하기 쉽게 작성할 수밖엔 없다. 

가장 좋은 것은 Jackson의 [애노테이션](http://www.baeldung.com/jackson-annotations)을 적절히 활용하여 로직의 수정 없이 serialization/deserialization 과정에서의 포맷만 변화를 주는 것이다. 하지만 애노테이션을 아무리 단다 한들 DTO를 작성하는 데 있어 Java가 갖는 구조적 한계점들 - getter/setter의 작성, 필드 선언의 장황함, 생성자 오버로딩 등.... - 을 넘을 수는 없을 것이다. 

#### We Go Kotlin!

해결책은 단 하나, **코틀린을 도입하는 것이다.**

Spring은 이미 Kotlin을 광범위하게 지원하고 [있다.](https://www.youtube.com/watch?v=SlBRce-aBOc) 물론 당장 비즈니스 로직에 코틀린을 적용할 만큼 코틀린에 숙련도가 높지도 않고, 무엇보다 협업을 해야하는 경우 동료 개발자들에게 Kotlin으로 이주하라고 강요하는 것은 아주 고약한 일이 될 것이다. 

나는 유연하고도 빠른 DTO의 작성을 위한 용도로만, 아주 제한적으로 Kotlin을 프로젝트에 적용하기로 마음먹었고, 이를 위해 Kotlin의 다른 부분은 학습하지 않고 data class 의 개념만 익혀 바로 프로젝트에 적용해봤다.

#### Data Class

Data class는 자바에는 없는 코틀린의 차별화된 기능으로 DTO, VO를 작성하기에 좋은 클래스다. 

Data class는 변수 var 대신 값 val 을 가지며 아래와 같이 구현이 가능하다. 

```kotlin
data class NewBookDocument(
        val title : String = "",
        val author : String = "",
        val description : String = "",
        val isbn : String = "",
        val category : String? = "",
        val location : String? = ""
)
```

필드와 생성자를 **동시에 선언**하였고, getter/setter도 구현할 필요 없다. **Lombok 사용한 게 아니다!**

같은 DTO를 Java로 작성했다면....

```java
public class NewBookDocument { 
    
    private String title;
    private String author;
    private String description;

    (...)


    public String getTitle() {
        return this.title;
    }

    @Override
    public String toString(){
        (...)
    }

    (...)
}
```

**아. 피곤하다.**

Kotlin의 Data class 개념에 대한 이해를 위해서는 [이 레퍼런스](http://kotlinlang.org/docs/reference/data-classes.html)를 참고할 것을 권한다. 

Data Class는 우리가 Java 개발 시에 사용했던 lombok 애노테이션 거의 대부분, @EqualsAndHashcode - @ToString 과 같은 애노테이션들을 사용할 필요 없이 자동으로 구현해준다. 

#### Jackson과 삽질하기 

한가지 아쉬운 점은 Kotlin의 data class에서 생성자를 정의할 때, **애노테이션을 직접 붙여주지 못한다는 점**이다. 

말이 어려운데 *아주 쉬운 얘기다*. 아래의 사례를 보며 같이 생각해보자. 

```kotlin
data class FooDto(
    val name: String = "",
    val age: Int = 0
)
```

까탈스러운 프론트엔드 개발자의 요구로 인해 JSON으로 serialize 시에 필드 이름을 FooName이라고 매핑해야 하는 상황이라고 가정해보자. 

```json
{"FooName" : "seulgi", "age": 24}
```

Java에서 하던 대로 하자면 자연스럽게 이렇게..... 시도해볼 것이다. (**적어도 나는 그랬다.**)

```kotlin
data class FooDto(
    @JsonProperty("FooName")
    val name: String = "",
    val age: Int = 0
)
```

접근방식 자체가 틀린 것은 아니다. Kotlin 파일에서도 Java에서 온 애노테이션들을 전부 사용할 수 있다. 그러나 우리는 Kotlin data class의 작동 방식에 대해 이해할 필요가 좀 있다. 

만약 상기한 것과 같이 애노테이션을 선언한다면 엄밀히 말하면 우리는 Java에서 했던 것처럼 field나 getter를 애노테이트 한 것이 아닌, **생성자의 파라미터에 애노테이트한 것**과 같다. Kotlin도 당연히 이렇게 이해한다. 

저 애노테이션을 객체의 field에서 사용하려면, 혹은 getter에서 사용하려면 아래와 같이 구현하면 된다. 

```kotlin
data class FooDto(
    @field:JsonProperty("FooName")val name: String = "",
    val age: Int = 0
)
```

```kotlin
data class FooDto(
    @get:JsonProperty("FooName")val name: String = "",
    val age: Int = 0
)
```

Java에서와 마찬가지로 deserialization 결과물은 같을 것이다. 이제 의도대로 Jackson 애노테이션이 작동한다! 

#### 아직 남은 문제점 

**Kotlin 클래스들에서 Lombok이 구현해준 메소드들을 사용할 수 없다.** 심각한 문제다. 

도메인 클래스 대부분에서 lombok이 만들어준 Builder 패턴 구현체들을 사용하는 나로써는 엄청난 문제가 아닐 수 없다. 

이는 컴파일 타임에서 Kotlin이 Java보다 먼저 컴파일되기 때문에, Java의 컴파일된 byte code 레벨에서 구현체를 만드는 lombok을 사용할 수 없는 것이다. 실제로 mvn compile을 수행해보면 Kotlin plugin들이 먼저 도는 것을 확인할 수 있다. 

이 문제를 해결하기 위해선 build configurations를 통해 Kotlin이 Java보다 나중에 컴파일되게 구현하면 되는데 이러면 Java에서 Kotlin을 보지 못하는 더 중대한 문제(...)가 발생하므로, 번거롭더라도 Java로 작성한 도메인 로직에 Kotlin 기반 DTO를 POJO로 바꿔주는 로직을 작성하는 것이 가장 좋다. 

**번거롭긴 해도 Java로 작성한 DTO와 씨름하는 것보다는 훨씬 나을 것이라 장담한다.**

이 문제에 대해선 [다음 문서](https://stackoverflow.com/questions/35517325/kotlin-doesnt-see-java-lombok-accessors)를 참고해보자. 
