---
title: Enum을 사용해 실수를 줄인다.
date: 2017-12-22 12:34:15
tags:
    - Java
    - enum
    - clean-code
    - Spring
---

### Enum을 적극적으로 사용해보자. 

Enum은 enumerated에서 따온 말로, 우리말로 옮기믄 "열거형" 이라고들 많이 하는 모양이다. 

개인적으로 "열거형" 이란 표현은 enum의 본질을 제대로 담고 있지 못하다고 생각해서 그리 바람직한 번역이라고는 생각되지 않는다. 또 Swift에서도 enum이 있는 것 같은데 그쪽 친구들은 이 괴상한 단어를 "이늄" 이라고 읽는다. 

**이늄 이라니, 얼마나 세련된 발음인가?** 애석하게도 우리 Java 진영에서는 대다수의 개발자들이 이걸 "이넘" 이라고 주로 읽는 것 같고 "에넘" 이라고 하시는 분도 간혹가다가 본다. enum의 어원인 enumerated라는 영단어의 밞음을 살펴보면 Swift쪽의 발음이 더 간지나는 것 뿐만 아니라 어원에도 충실한 발음인 것 같다. 

enum을 어떻게 선언하는지, 어떻게 값을 관리하는지에 대한 개념 설명은 **이 글에서 다루지 않겠다.** 내가 생각하기에 (내가) enum을 더 많이 사용해야 하는 이유, 개발 상의 용이한 점 및 주의점들에 대해 서술해 보고, 마지막에 지금 진행중인 프로젝트에서 내가 어떻게 enum을 활용하고 있는지에 대해 간단하개 코드 중심으로 공유하고 글을 마치고자 한다. 

#### 왜 우리는 enum을 사용해야 하는가? 

예제 코드로 시작해보자. 

```Java
public class RedVelvet {
    
    private List<String> members = new ArrayList<>();

    (...)

    members.add("슬기");
    members.add("웬디");
    members.add("조이");
    members.add("예리");
    members.add("아이린");

    //standard getter/seters, hashCode(), toString() follows....
}
```

국내 최고의 인기 걸그룹 레드벨벳의 멤버 정보를 담고 있는 이 클래스는 다섯 멤버들의 이름을 String으로 하드코딩하여 사용하고 있다. 자. 그러면 다른 객체에서 멤버들의 이름 정보를 가져와서 사용해 볼까.....

```Java
public class RedVelvetConcert {

    (...)
    
    public void printConcertInfo() {
        String msg = String.format("이번 차례는 %s의 솔로 무대입니다!" , RedVelvet.getMembers.get(1));
    }
   
}
```

어떤 느낌이 드는가? 우리가 RedVelvetConcert 클래스를 설계할 때, 멤버 정보를 담고있는 RedVelvet.getMembers()의 반환값이 슬기, 웬디, 조이, 예리, 아이린의 순서대로 정렬되어 있음을 보장할 수 있을까? 

RedVelvetConcert를 설계한 사람이 속세와 인연을 끊고 사는 열헐 개발자인 경우엔 어떻게 할까? 그는 이 코드를 작성하며 레드벨벳의 멤버 정보에 대해 **IDE로부터 전혀 힌트를 얻지 못한다.**

RedVelvet을 enum으로 바꿔주면 어떨까? 아래와 같이 말이다. 

```Java
public enum RedVelvet {
    SEULGI("슬기"),
    WENDY("웬디"),
    JOY("조이"),
    YERI("예리"),
    IRENE("아이린");

    private String name;
    
    RedVelvet(String name) {
        /*많은 개발자들이 enum의 생성자에 private 접근 제어자를 붙이는 것을 알고 있다.
        하지만 굳이 그러지 않아도 된다.*/

        this.name = name;
    }

    public String getName() {
        return this.name;
    }
} 
```

자, 이제 printConcertInfo()도 다시 작성해보자.

```Java
public void printConcertInfo() {
    String msg = String.format("이번 차례는 %s의 솔로 무대입니다!" , 
    RedVelvet.WENDY.getName());
}
```

생 String을 뽑아와서 붙일 때보다 훨씬 간결하고, 무엇보다 **누구나 이해할 수 있는 코드가 되지 않았나?** Wendy를 "웬디" 라고 발음한다는 정도만 알면 레드벨벳을 잘 모르는 사람이라도 쉽게 이 코드를 유지보수할 수 있을 것이다. 

---
#### JPA Entity에서 Enum을 활용해보자.

**JPA로 관리하는 Model 객체에서도 Enum을 사용할 수 있다.** 아래와 같이 구현하면 된다. 

```Java

@Entity
public class RedVelvetMember {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    (...)

    @Enumerated(EnumType.STRING)
    private RedVelvet redVelvet;
}
```

@Enumerated 애노테이션이 EnumType이라는 또다른 enum을 field로 갖고 있는 것을 볼 수 있는데 개발자에게 주어진 옵션은 두 가지다. 
>EnumType.STRING = enum의 .name() 값을 칼럼에 입력한다 
>
>(e.g. RedVelvet.SEULGI => SEULGI 라고 칼럼에 입력됨)
>
>EnumType.ORDINAL = enum의 순서가 INT로 칼럼에 입력된다 
>
>(e.g. RedVelvet.IRENE => 다섯번째 요소이므로 5로 입력됨)

enum을 한 번 구현해놓고 **절대로 내용이나 순서를 바꿀 생각이 없다면** ORDINAL로 써도 된다. *근데 그런 경우가 있을까?* 

실제로 많은 JPA 책에서도 .STRING 을 사용할 것을 강력 추천하고 있다. 왠만하면 STRING 쓰자. 

---
#### 더 읽어보기 

[우아한형제들 개발자가 enum 적용시켜 본 이야기 - Java Enum 활용기](http://woowabros.github.io/tools/2017/07/10/java-enum-uses.html)

