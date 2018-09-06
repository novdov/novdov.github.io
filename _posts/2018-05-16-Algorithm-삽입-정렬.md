---
title: '[Algorithm] 삽입 정렬'
date: 2018-05-16 23:44:54
categories:
- Algorithm
tags:
- algorithm
---

> 참고: 프로그래밍 대회에서 배우는 알고리즘 문제 해결 전략 (구종만 저)

이 글의 코드는 원저의 C++ 코드를 파이썬으로 옮긴 코드입니다.

### 삽입 정렬의 작동 방식

- 전체 배열 중 정렬되어 있는 부분 배열에 새 원소를 끼워넣는 일을 반복
- 배열을 순회하면서 그 앞의 원소가 해당 원소보다 크면 두 원소의 위치를 교환
- 전체 시간 복잡도는 역순으로 정렬된 배열이 주어질 경우 `for`문에서 $O(N)$, `while`문에서 $O(N)$이 되기 때문에 $O(N^2)$

```python
def insertion_sort(array):
    for i in range(len(array)):
        j = i
        while (j > 0) and (array[j-1] > array[j]):
            # 원저의 C++에서는 swap(array[j-1], array[j]);
            array[j-1], array[j] = array[j], array[j-1] 
            j -= 1
    return array

array = [1, 4, 7, 11, 5, 6, 12, 9]
print(insertion_sort(array))
# [1, 4, 5, 6, 7, 9, 11, 12]
```

