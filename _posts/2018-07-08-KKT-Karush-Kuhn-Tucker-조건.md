---
title: KKT (Karush-Kuhn-Tucker) 조건
description: 최적화에서 사용되는 라그랑주 승수법과 KKT 조건에 대해 알아봅니다.
category:
- MachineLearning
tags:
- python
- machine learning
---

## 라그랑주 승수법 (Lagrange Multiplier)

등식 제한 조건이 있는 최적화 문제는 라그랑주 승수법을 사용해 최적화할 수 있다. 라그랑주 승수법에서는 $f(x)$ 가 아닌

$$h(x, \lambda) = f(x) + \sum_{j=1}^M\lambda_j g_j(x)$$ 

라는 함수를 목적함수로 보고 최적화한다. $h$ 는 독립 변수 $\lambda$ 가 추가되었으므로 다음 조건을 만족해야 한다.

$$\begin{eqnarray}\dfrac{\partial h(x, \lambda)}{\partial x_1} &=& \dfrac{\partial f}{\partial x_1} + \sum_{j=1}^M \lambda_j\dfrac{\partial g_j}{\partial x_1} = 0 \\\ \dfrac{\partial h(x, \lambda)}{\partial x_2} &=& \dfrac{\partial f}{\partial x_2} + \sum_{j=1}^M \lambda_j\dfrac{\partial g_j}{\partial x_2} = 0 \\\ \vdots && \\\ \dfrac{\partial h(x, \lambda)}{\partial x_N} &=& \dfrac{\partial f}{\partial x_N} + \sum_{j=1}^M \lambda_j\dfrac{\partial g_j}{\partial x_N} = 0 \\\ \dfrac{\partial h(x, \lambda)}{\partial \lambda_1} &=& g_1 = 0 \\\ \vdots & & \\\ \dfrac{\partial h(x, \lambda)}{\partial \lambda_M} &=& g_M = 0 \end{eqnarray}$$

 

위의 $N+M$ 개의 연립방정식을 풀면 $N+M$ 개의 미지수 $x_1, x_2, \ldots, x_N, , \lambda_1, \ldots , \lambda_M$ 를 구할 수 있는데, 여기에서 $x_1, x_2, \cdots, x_N$ 이 제한 조건을 만족하는 최솟값 위치를 나타낸다.



## KKT 조건

$g(x) \le 0$ 이라는 부등식 제한 조건이 있는 최적화 문제에서도 마찬가지로 라그랑주 승수법과 마찬가지로 

$$h(x, \lambda) = f(x) + \sum_{j=1}^M\lambda_j g_j(x)$$

를 목적함수로 보고 최적화한다.

단, 이 경우 최적화 필요 조건은 등식 제한 조건의 경우와는 달리 KKT(Karush-Kuhn-Tucker) 조건으로 부르며, 다음의 3개의 조건으로 이루어진다.

- 모든 독립 변수에 대한 미분이 0: $\dfrac{\partial h(x, \lambda)}{\partial x_i} = 0$
- 모든 라그랑주 승수와 부등식의 곱이 0: $\lambda_j \cdot \dfrac{\partial h(x, \lambda)}{\partial \lambda_j} = \lambda \cdot g_j = 0$
- 음수가 아닌 라그랑주 승수: $\lambda_j \ge 0$





