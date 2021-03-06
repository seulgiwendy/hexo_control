---
title: 아침 CS - (2) 트리 순회
date: 2018-04-09 09:55:06
categories:
    - data_structure
    - algorithm
    - morning_cs
---

### 트리 순회

트리 순회는 트리 자료구조를 탐색해 데이터를 찾는 순회 방법을 말하는 것이다.

**후위 순회, 중위 순회, 전위 순회, 레벨 순회** 의 큰 네가지로 이뤄진다. 

---

#### 예제 트리

아래의 그림으로 네 가지의 순회법을 설명한다. 

![이진 트리](https://www.geeksforgeeks.org/wp-content/uploads/2009/06/tree12.gif)

---

#### 후위 순회

**내가 제일 좋아하는 순회라서 가장 앞에 소개한다.** 

> 왼쪽 오른쪽 부모 

간단하다. 

먼저 왼쪽 노드의 왼쪽 자식(무조건 왼쪽, 좌파다)을 방문하고, 오른쪽 노드, 그리고 부모 순으로 방문하면 된다. 

*부모는 무조건 맨 나중이다.* 4 - 5 - 2 - 3 - 1 순서로 방문. 

---

#### 중위 순회

> 왼쪽 부모 오른쪽 

어떻게보면 가장 상식적으로 추론할 수 있는 순회법이다. 

가장 왼쪽의 노드를 먼저 방문한 다음 슬래시를 치듯이 방문하면 된다. 

4 - 2 - 5 - 1 - 3.

---

#### 전위 순회

> 부모 왼쪽 오른쪽 

부모님을 공경하는 순회법이다. 

그러나 자식 노드를 방문할 땐 왼쪽부터 방문해야 하는 건 같다. 

**오해하면 안되는게 재귀적으로 왼쪽을 쭉 탐색한 다음 오른쪽 노드로 넘어가는 것이다.** 즉 왼쪽 노드가 자식을 갖고 있으면 그 자식들 중 왼쪽 노드를 또 탐색해야 한다. 

1 - 2 - 4 - 5 - 3.

---

#### 레벨 순회 

*부야!*

> 같은 층부터 방문 

나와 같은 레벨의 노드들부터 방문, 하나씩 레벨을 내려가며 왼쪽 노드부터 읽는다.

1 - 2 - 3 - 4 - 5.

---

#### 추가로 학습할 점

* 각 순회를 코드로 구현해보면 좋겠다. 

---

#### 트리 순회를 실생활에 적용하는 법

명절날 친정부터 갈지 시댁부터 갈지 정할때 활용해보면 좋다. 

**여보, 이번 추석은 후위 순회 할까?**

---

#### 연습문제 풀어보기

[링크](https://jiwondh.github.io/2017/10/15/tree_traversal/#-%EC%A4%91%EC%9C%84-%EC%88%9C%ED%9A%8C-inorder-traversal)

