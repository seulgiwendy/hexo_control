---
title: 알고리즘 타임 - (2) N개의 최소공배수
date: 2018-04-08 08:38:56
categories:
    - algorithm
    - programmers_exercise
---

### N개의 최소공배수

두 수의 최소공배수(Least Common Multiple)란 입력된 두 수의 배수 중 공통이 되는 가장 작은 숫자를 의미합니다. 예를 들어 2와 7의 최소공배수는 14가 됩니다. 정의를 확장해서, n개의 수의 최소공배수는 n 개의 수들의 배수 중 공통이 되는 가장 작은 숫자가 됩니다. nlcm 함수를 통해 n개의 숫자가 입력되었을 때, 최소공배수를 반환해 주세요. 예를들어 [2,6,8,14] 가 입력된다면 168을 반환해 주면 됩니다.

---

#### 풀이

두 수의 최소공배수는 두 수의 최대공약수와 각 수를 최대공약수로 나눈 몫의 곱이다.

```
LCM = GCD * (Num1 / GCD) * (Num2 / GCD)
```

사실 Python에는 두 수의 최대공약수를 구하는 내장 함수가 있지만 **사용하지 않기로 한다**

N개의 원소를 갖는 집합에서 최소공배수는 (N - 1) 까지의 최소공배수와 N 의 최소공배수를 구하면 된다. 

```
NLCM = LCM(N, LCM( ~ N -1))
```

---

#### 코오드

```Python
def nlcm(num):
    return lcm_recurse(num)

def lcm_recurse(array):
    if len(array) == 1:
        return array[0]

    if len(array) == 2:
        return find_lcm(array[0], array[1])

    return find_lcm(array[-1], lcm_recurse(array[:-1]))

def find_lcm(num1, num2):
    gcd = find_gcd(num1, num2)

    return int(gcd * (num1 / gcd) * (num2 / gcd))

def find_gcd(num1, num2):
    current_divider = 0

    if num1 > num2:
        current_divider = num2
    else:
        current_divider = num1

    while(True):
        if num1 % current_divider == 0 and num2 % current_divider == 0:
            break
        current_divider -= 1

    return int(current_divider)
```
---
#### 더 나은 풀이

다른 사람들의 풀이를 보니 `gcd()` 빌트인 함수를 사용하신 분들이 많았다 

알고리즘 문제에서 이런 식의 풀이는 의미가 없다고 생각하지만, 입사용 코딩 테스트 등에서 이런 라이브러리의 사용이 불이익이 되지 않는다면 앞으로 적극적으로 사용하는 것을 검토해 볼 만 하겠다. 

눈에 띄는 풀이는 없었지만 *정말 우아하게 최대공약수를 구하는 분* 이 있어 코오드를 여기에 살짝 옮겨 본다. 

```Python
def gcd(a, b):
    if b == 0:
        return a
    return gcd(b, a % b)
```
---
#### 느낀 점

* 재귀 호출을 한 번에 내가 생각한대로(...)사용할 수 있었다. 실력이 좀 늘었나?
* 메소드(함수) 분리를 너무 많이 하는 것 같다. 
    * 실무에선 좋은 습관이다. 
    * 알고리즘 풀이에선?


