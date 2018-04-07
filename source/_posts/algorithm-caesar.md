---
title: 알고리즘 타임 - (1) 시저 암호
date: 2018-04-07 21:32:17
categories:
    - algorithm
    - programmers_exercise
---

### 시저 암호

어떤 문장의 각 알파벳을 일정한 거리만큼 밀어서 다른 알파벳으로 바꾸는 암호화 방식을 시저 암호라고 합니다.

A를 3만큼 밀면 D가 되고 z를 1만큼 밀면 a가 됩니다. 공백은 수정하지 않습니다.

보낼 문자열 s와 얼마나 밀지 알려주는 n을 입력받아 암호문을 만드는 caesar 함수를 완성해 보세요.

* “a B z”,4를 입력받았다면 “e F d”를 리턴합니다.

---

#### 풀이

기본적으로 [아스키 코드표](http://egloos.zum.com/littletrue/v/4781098) 를 숙지하고 있으면 어렵지 않게 해결할 수 있는 문제다. 

127가지나 되니 숙지하진 말고(...) 이런 개념이 있다, 를 알고 필요할 때 찾아보면 된다. **정규표현식은 외우면 외울수록 좋지만 이건 잘 모르겠다.** 다만 알파벳 정도는 몇 번 부터 몇 번까지인지를 알아두는 것이 좋겠다. 

* 알파벳 소문자`(a ~ z)` : 97 ~ 122
* 알파벳 대문자`(A ~ Z)` : 65 ~ 90

**범위를 초과하는 암호 인덱스에 대해서**

알파벳은 26글자이므로 26으로 나눈 나머지를 취하면 된다. 

예를 들어 알파벳을 한 글자씩 134스텝을 뛴다 해도 결국 다섯바퀴를 돌아 네 번째 인덱스, 즉 D를 취하게 되는 것이다. 

알파벳의 중간에서부터 출발해 알파벳의 범위를 넘는다면 알파벳의 갯수만큼 인덱스를 빼 주면(-26) 그만이다. 

---

#### 코오드 

```python
def caesar(s, n):
    string_array = split_string(s)
    encrypted_array = return_encrypted(string_array, n)

    result = str.join("", encrypted_array)
    return result

def split_string(string):
    return list(string)

def return_encrypted(array, n):
    encrypted_char = []
    for char in array:
        origin_value = ord(char)
        next_value = origin_value + (n % 26)

        if(origin_value == 32):
            encrypted_char.append(chr(32))
            continue

        if(origin_value > 64 and origin_value < 91):
            if next_value > 90:
                next_value = next_value - 26

        if(origin_value > 96 and origin_value < 123):
            if next_value > 122:
                next_value = next_value - 26

        print(next_value)
        encrypted_char.append(chr(next_value))

    return encrypted_char
```
---

### 느낀 점

* 파이썬의 아스키 코드 함수 : `chr(), ord()`를 배움
    * 문자를 코드로 : `ord()`
    * 코드를 문자로 : `chr()`

* 아스키 코드표, 좀 외워두면 도움이 될까?

