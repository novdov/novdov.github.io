---
title: '[Algorithm] 시간 복잡도와 수행 시간'
date: 2018-05-15 01:14:31
categories:
- Algorithm
tags:
- algorithm
---

> 참고: 프로그래밍 대회에서 배우는 알고리즘 문제 해결 전략 (구종만 저)

이 글의 코드는 원저의 C++ 코드를 파이썬으로 옮긴 코드입니다.



1차원 배열에서 연속된 부분 구간 중 그 합이 최대인 구간을 찾는 문제는 여러 알고리즘으로 해결할 수 있다. 시간 복잡도가 서로 다른 여러 알고리즘을 구현하고 각 알고리즘의 수행 시간이 어떻게 다른지 확인해본다.

테스트 배열로는 $A = \begin{bmatrix} -7, 4, -3, 6, 3, -8, 3, 4 \end{bmatrix}$ 을 사용하며, 해답은 $\begin{bmatrix}4, -3, 6, 3 \end{bmatrix}$ 일 때 10이다.



### 1. 실행 시간 $O(N^3)$

```python
A = [-7, 4, -3, 6, 3, -8, 3, 4]

# N**2개의 후보 구간을 검사하고
# 각 구간의 합을 구하는데 N시간이 걸리는 알고리즘
def inefficient_max_sum(array):
    n = len(array)
    max_sum = 0
    for i in range(n):
        for j in range(n):
            _sum = 0
            for k in range(i, j):
                _sum += array[k]
            max_sum = max(max_sum, _sum)
    return max_sum

print(inefficient_max_sum(A))
# 10
```



### 2. 실행 시간 $O(N^2)$

```python
def better_max_sum(array):
    n = len(array)
    max_sum = 0
    for i in range(n):
        _sum = 0
        # 구간 array[i, j]의 합을 구한다
        for j in range(i, n):
            _sum += array[j]
            max_sum = max(max_sum, _sum)
    return max_sum

print(better_max_sum(A))
# 10
```



### 3. 실행 시간 $O(N\log{N})$

분할 정복과 탐욕 알고리즘을 사용하면 실행 시간을 $O(N\log{N})$으로 줄일 수 있다. 1) 입력받은 배열을 반으로 잘라 왼쪽 배열과 오른쪽 배열로 나눈다. 2) 최대 합 부분 구간은 두 배열 중 하나에 속해 있을 수도 있고, 두 배열 사이에 걸쳐 있을 수도 있다. 3) 이 때 각 경우의 답을 재귀와 탐욕법을 이용해 계산하면 분할 정복 알고리즘이 된다.

```python
# 분할 정복, 탐욕 알고리즘 사용
def fast_max_sum(array, lo, hi):
    n = len(array)
 
    # 길이가 1이면 그 값을 그대로 리턴
    if lo == hi:
        return array[lo]
    
    # 배열을 반으로 나눔
    mid = (lo + hi) // 2
    # 왼쪽/오른쪽 배열의 합, 합을 저장할 초깃값
    left, right, _sum = 0, 0, 0
    
    # 두 부분에 모두 걸쳐 있는 최대 합 구간을 찾는다.
    # 이 구간은 array[i, mid]와 array[mid+1, j] 형태를 갖는 구간의 합으로 이루어진다.
    # array[i, mid] 형태를 갖는 최대 구간을 찾는다.
    for i in range(mid, 0, -1):
        _sum += array[i]
        left = max(left, _sum)
    
    # array[mid+1, j] 형태를 갖는 최대 구간을 찾는다.
    _sum = 0
    for j in range(mid+1, hi):
        _sum += array[j]
        right = max(right, _sum)
    
    # 최대 구간이 두 배열 중 하나에만 속해 있는 경우의 답을 재귀로 찾는다.
    single = max(fast_max_sum(array, lo, mid), 
                 fast_max_sum(array, mid+1, hi))
    
    # 두 경우 중 최대치를 리턴
    return max(left + right, single)

print(fast_max_sum(A, 1, 5))
# 10
```



### 4. 실행 시간 $O(N)$

동적 계획법을 사용하면 선형 시간에 문제를 풀 수 있다. 배열 $A[i]$를 오른쪽 끝으로 갖는 구간의 최대 합을 리턴하는 함수 $max  At(i)$를 정의해보자. 이 때, $A[i]$ 에서 끝나는 최대 합 부분 구간은 항상 $A[i]$ 하나만으로 구성되어 있거나, $A[i-1] $을 오른쪽 끝으로 갖는 최대 합 부분 구간의 오른쪽에 $A[i]$ 를 붙인 형태로 구성되어 있음을 증명할 수 있다. 따라서 $maxAt()$ 를 다음과 같은 점화식으로 표현할 수 있다.

$$maxAt(i) = \max(0, maxAt(i-1)) + A[i]$$

```python
def fastest_max_sum(array):
    n = len(array)
    max_sum, psum = 0, 0
    for i in range(n):
        psum = max(psum, 0) + array[i]
        max_sum = max(psum, max_sum)
    return max_sum

print(fastest_max_sum(A))
# 10
```



1초 안에 처리할 수 있는 최대 입력 크기는 각 알고리즘에 따라 다음과 같이 변한다.

- $O(N^3)$: 크기 2560인 입력까지를 1초 안에 풀 수 있다. $2560^3$은 대략 160억이다.
- $O(N^2)$: 크기 40960인 입력까지를 1초 안에 풀 수 있다. $40960^2$은 대략 16억이다.
- $O(N\log{N})$: 크기가 대략 2천만인 입력까지를 1초 안에 풀 수 있다. 이 때 $N\log{N}$은 대략 5억이다.
- $O(N)$: 크기가 대략 1억 6천만인 입력까지 1초 안에 풀 수 있다.