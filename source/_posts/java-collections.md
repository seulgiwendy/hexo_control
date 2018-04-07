---
title: JCF의 자료구조와 시간복잡도를 알아보자. - (1) List 편
date: 2018-04-07 15:01:15
categories:
    - java
    - data_structure
    - algorithm
---

### Java Collections Framework 구현체들의 자료구조
---

#### List

* List 가기전에 벡터(Vector) 이야기 잠깐

**Vector 란?**

* JDK 1.0 부터 존재하던 표준
* ArrayList 출시되기 이전에 즐겨 사용
* 항상 Thread-safe함. (무조건 동기화)

    * ArrayList와의 성능 차이를 만드는 요인이자 요즘 Vector를 안 쓰는 이유.
    * 정말 Thread-safe한 리스트가 필요하다면 Synchronized List 등을 사용하는게 낫다. 


#### List 인터페이스의 구현체들

* AbstractList
* AbstractSequentialList
* ArrayList(!)
* AttributeList
* CopyOnWriteArrayList
* LinkedList(!)
* RoleList
* RoleUnresolvedList
* Stack
* Vector 

**시간 복잡도**

`size(), isEmpty()` : `O(1)`

`remove(), contains(), iterate()` : `O(n)`

---

#### ArrayList

즐겨 사용하는 어레이리스트를 뜯어보자. 

*초기 생성*

```java

public class ArrayList {

    private transient Object[] elementData;

    (...)

    public ArrayList(int initialCapacity) {
    super();
    if (initialCapacity < 0) {
        (예외 처리 로직)
    }
    this.elementData = new Object[initialCapacity];
    }

    public ArrayList() {
        this(10); //크기를 명시하지 않으면 length 10의 배열 생성.
    }
}
```

* 내부적으로 transient 키워드를 사용하는 Object 배열 사용.
* 크기를 생성자로 명시할 수 있음. 명시하지 않는다면 크기는 10
* 자료의 빈번한 삭제와 삽입에는 불리함.
* 배열의 인덱스 값을 이용해 한 번에 원하는 값을 찾아올 수 있음. 

---

#### LinkedList

링크드리스트는 링크드리스트다. **더 이상의 설명이 필요한가?**

*뷁*

**초기 생성**

```java
public class LinkedList<E> extends AbstractSequentialList<E> (...){
    
    private transient Entry<E> header = new Entry<E>(null, null, null);
    private transient int size = 0;

    public LinkedList() {
        header.next = header.previous = header;
    }

    public LinkedList(Colleciton<? exttends E> c) {
        this();
        addAll(c); //생성자로 컬렉션을 전달하면 모든 원소를 링크드리스트에 담아 준다.
    }
}
```

#### 알아둘 점

* 링크드리스트 자체에 대한 이해를 하자. 
* 링크드리스트는 값과 다음 값의 위치를 갖고 있는 자료구조. 

--- 






