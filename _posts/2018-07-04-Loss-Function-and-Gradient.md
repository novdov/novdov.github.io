---
title: Loss Function and Gradient
date: 2018-07-04 15:34:45
description: 최적화 문제와 Loss Function
category:
- MachineLearning
tags:
- python
- machine learning
---

> 그 동안 loss function과 gradient를 공부한 뒤 수학적 의미를 자세히 복습하지 않아서 다시 해당 내용을 복습!



## 최적화 문제와 Loss Function

최적화 문제는 모수를 입력으로, 예측 오차를 출력으로 하는 함수 $f$ 의 값을 최소화하는 $x$ 의 값 $x^{\ast}$ 를 찾는 것이다.

$$x^{\ast} = \arg \underset{x}{\min} f(x)$$

이 때 최소화 하려는 함수를 Objective/Cost/Loss Function 등으로 부른다.



## 수치적 최적화 (Numerical Optimation)

반복적 시행 착오로 최적화 필요조건을 만족하는 $x^{\ast}$ 를 찾는 방법을 수치적 최적화라고 한다. 수치적 최적화는 함수 위치가 최적점이 될 때까지 가능한한 적은 횟수만큼 $x$ 의 위치를 옮기는 방법이다. 이는 두 가지로 구성된다.

- 현재 위치 $x_k$ 가 최적점인지 판단하는 알고리즘
- 어떤 위치를 시도한 뒤, 다음 번에 시도할 위치 $x_{k+1}$ 을 찾는 알고리즘



### 기울기 필요 조건

현재 시도하고 있는 위치가 최소점인지 알아내기 위해 미분을 이용한다. $x^{\ast}$ 가 최소점이 되기 위해서는 $x^{\ast}$ 에서의 함수의 기울기 $\dfrac{df}{dx}$ 의 값이 0이어야 한다. 이를 기울기 필요 조건이라 한다.

즉,

$$\nabla f = \begin{bmatrix}\dfrac{\partial f}{\partial x_1} \\\ \dfrac{\partial f}{\partial x_2} \\\ \vdots \\\ \dfrac{\partial f}{\partial x_n} \end{bmatrix} = 0$$

을 만족해야 한다.

이 때 함수 $f$ 의 **편미분 값을 그래디언트(Gradient)** 라고 한다.



### SGD (Steepest Gradient Descent)

SGD 방법은 현재 위치에서의 기울기 $g(x_k)$ 만을 이용해 다음에 시도할 위치를 알아내는 방법이다.

$$x_{k+1} = x_k - \mu \nabla f(x_k) = x_k - \mu g(x_k)$$ 

현재 위치 $x_k$ 에서 기울기가 음수이면 앞으로 진행하고, 양수이면 뒤로 진행해 점점 낮은 위치로 이동한다. $g(x_k) = 0$ 이면 최소점에 도달한 것이므로 더 이상 위치를 옮기지 않는다.



간단히 정리하면 SGD는 현재 위치에서 (initial point) 가장 함수값이 작아지는 방향을 찾아서 (gradient) 이동하는 (next position) 알고리즘이다.