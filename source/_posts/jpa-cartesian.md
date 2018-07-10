---
title: JPA N + 1 문제를 해결하자. 
date: 2018-06-16 10:57:31
categories:
    - jpa
    - database
---

### JPA N + 1 문제를 해결하자.

지난 겨울 프로젝트를 진행하며 일대다 관계를 많이 갖는 엔티티 클래스를 작성할 일이 있었다. 여느 때처럼 JPA의 도움을 받아 어렵지 않게 작성했다고 생각했는데, 데이터베이스 관리와 데브옵스 역할을 담당하고 있던 팀원이 난처한 표정을 짓는 것을 볼 수 있었다. 

**엔티티 한 개 조회할 때마다 엄청난 양의 쿼리가 발생한 것이다.** 

처음에는 이 문제에 대해 정확히 이해하지 못하고 자식 엔티티의 자료구조를 바꾼다던지, named query를 사용한다던지 하는 피상적인 해결책만을 생각해낼 수밖에 없었다. SQL의 관점에서 생각해보면 자식 엔티티는 당연히 INNER JOIN 과 같은 키워드로 한 번에 빼오면 되는 문제였는데 이렇게 많은 쿼리가 발생한다는 것을 이해하지 못했기 때문이었다. 

그러나 JPA에 대해 학습하면 할수록 이런 동작은 단순한 우연이 아닌 필연적인 부분이란 것을 알게됐고 이런 이슈를 최적화하는 것이 **역량있는 개발자의 모습** 이 아닐까 하고 생각하게 됐다. 더불어 상당수 조직과 초보 개발자들이 JPA 도입을 꺼리는 이유이기도 하다는 걸 알게 됐다. 

#### 언제 발생하는가? 

다음의 예제 엔티티 클래스들과 함께 설명을 진행해보겠다. 

마스터와 학생은 일대다 관계를 갖고 있다. 한 마스터가 여러명의 학생을 관리하고 있고, 양방향 연관관계가 수립되어 있다. 

```java

@Entity
public class Master {

    @Id
    @GeneratedValue
    private long id;

    @OneToMany
    private List<Student> students = Lists.newArrayList();

    ...
}
```

```java

@Entity
public class Student {

    @Id
    @GeneratedValue
    private long id;

    @ManyToOne
    private Master master;

    ...
}
```

아래는 마스터들을 조회할 수 있는 레포지토리 인터페이스이다. 

```java
public interface MasterRepository extends JpaRepository<Long, Master> {

    (JPA 레포지토리가 기본 제공해주는 메소드들이 당연히 포함되어 있다.)

}
```

이 상황에서 MasterRepository.findAll() 을 실행했을 때, 우리는 이런 쿼리가 날라가길 기대할 것이다.

```sql
SELECT * FROM MASTER
LEFT JOIN STUDENT
ON STUDENT.MASTER_ID = MASTER.ID
```

*현실은.....*

```sql
SELECT * FROM MASTER

SELECT * FROM STUDENT WHERE MASTER_ID = 0
SELECT * FROM STUDENT WHERE MASTER_ID = 1
SELECT * FROM STUDENT WHERE MASTER_ID = 2
SELECT * FROM STUDENT WHERE MASTER_ID = 3

...
```

DB에 상당한 부하를 일으키게 된다. 

#### 왜 이러는가?

JPA가 객체를 조회할 때 SQL이 아닌 **JPQL의 생성과 파싱** 으로 출발하기 때문이다. JPQL이란 SQL을 추상화한 객체지향 쿼리 언어로써, 자바 코드에서 데이터베이스를 조회할 때 특정 SQL 방언이나 저장 엔진에 종속되지 않게 도와준다. 또한 자바 코드를 작성하며 조회할 수 없을만 한 데이터베이스의 테이블명이나 컬럼 이름 대신 POJO의 필드명을 쿼리에 사용할 수 있으므로 개발에 편의성을 제공해 준다. 

그리고 JDBC template의 Prepared Statement보다 한 단계 더 사용하기 편리한 지명 파라메터(named parameter) 기능을 제공한다. 말 그대로 쿼리 안에 변수를 대입할 수 있는 기능으로 잘 활용해 쿼리의 재사용성을 확보하여 조회 성능을 튜닝하고, SQL 주입 공격을 방어할 수 있다. 

언뜻 보면 SQL보다도 좋아보이는 외계인의 기술 같지만 **JPQL의 생성과 실행도 결국 SQL의 실행으로 귀결된다** 는 점을 기억해야 하겠다. 우리가 겪고 있는 N + 1 문제도 사실 여기서 출발한 것이다. 

*간단한 JPQL 시범*

모든 마스터 엔티티를 조회한다고 해보자. 

```sql
SELECT m from Master m
```

언뜻 보면 마스터의 모든 정보를 조회한 것 같지만 그렇지 않다. 애스터리스크가 있어야 할 자리에 객체의 별칭(alias)이 들어가 있는 점에 주목하자. 

```java
List<Master> masters = em.createQuery("select m from Master m", Master.class).getResultList();
```

모든 마스터를 불러온다. 아까 살펴본 것처럼 마스터 객체에는 List<Student> students 라는 필드가 있었다. 

상기한 쿼리가 실행된 시점에서 이 필드는 *아직 조회되지 않았다.* 

**지연 로딩**

객체에서 해당 필드에 대한 정보를 필요로하게 된 시점에, 지연 로딩이 발생하고, 여기서 모든 문제가 시작된다. 

```java
Master m = masters.get(0);
System.out.println(m.getStudents.size());
```

마스터 객체에 대한 조회는 이미 끝났는데 학생 객체를 다시 조회해야 하므로 JOIN 문을 사용할 수 없고, 마스터의 @Id 값만 가지고 일일이 SELECT문을 만들어 다시 조회해야 하는 부담이 생긴다. 

### 해결책 

**단순하게 생각하면 - EAGER 하게 로딩한다. (즉시 로딩)**

초보 개발자들이 흔히 하는 오해이고 이렇게 되면 모든 문제가 해결되리라 생각한다. 이렇게 엔티티 클래스를 고치는 것이다. 

```java
@OneToMany(fetch = FetchType.EAGER)
List<Student> students = Lists.newArrayList();
```

그러나 작성된 쿼리 메소드가 JPQL로 변환되고 다시 그 JPQL이 실행되는 Spring Data JPA의 특성상, 이런 수정은 **절대로 문제를 해결해주지 않는다.**

#### 두 번 생각하면 - join fetch 전략 사용

커스텀 레포지토리를 작성한 후 직접 쿼리 메소드에 사용할 JPQL을 명시해주면서, fetch 키워드를 추가해주는 것이다. 

커스텀 레포지토리의 작성은 이 글의 범위를 벗어나는 것 같아 다루지 않겠다. 아래와 같은 JPQL을 넣어주면 된다. 

```sql
select m from Master m join fetch m.students
```

이는 아래와 같은 SQL문으로 번역된다.

```sql
SELECT * FROM MASTER
LEFT JOIN STUDENT
ON STUDENT.MASTER_ID = MASTER.ID
```

*(실제 번역내용은 하이버네이트의 규칙을 따르므로 이와 상이하다)*

#### 자식 엔티티 조회에 조건이 걸린다면 - FetchMode.SUBSELECT

JPQL을 직접 작성해 영속성 콘텍스트에서 데이터를 조회하는 시점에 조건문이 걸린다면 이와 같은 구현을 해볼만 하다. 

```sql
select m from Master m where m.id > 10
```

학생을 관리하는 리스트 필드에 @Fetch(FetchMode.SUBSELECT) 가 걸려있다면 

```sql
SELECT * FROM STUDENT
WHERE STUDENT.MASTER_ID IN (
    SELECT ID FROM MASTER
    WHERE ID > 10
)
```

위와 같이 실행된다. 

### 마무리

N + 1과의 사투는 JPA 개발자라면 누구나 한번쯤 겪어보며 경험치를 올려야 할 이슈들 중에 하나다. 

물론 이 글에서 나는 단편적으로 N + 1 문제만을 해결하는 방법 몇 가지에 대해서만 설명한 것이고 실제 MVC계층 구조를 갖는 Spring 환경에서 이 해결책들을 적용하려면 Lazy Initialization Exception 문제를 비롯해서 JPA 객체의 생명주기에 따른 다양한 문제점들이 파생되어 나올 것이다(DETACHED 상태에서 지연로딩이 발생하지 않는다는 것을 기억하자). 

다음에 기회가 되면 이런 문제들에 대한 솔루션도 글로 정리해보고 싶다. 

여튼, N + 1 문제를 해결하려면 다음 두 가지를 잘하자. 

* 어지간해선 지연 로딩을 사용하자(기본값이 즉시로딩인 경우에도 애노테이션을 통해 지연 로딩을 강제하자)
* JOIN이 필요하다고 생각되는 시점에 인위적으로 JOIN을 걸어주자. 

끝. 




