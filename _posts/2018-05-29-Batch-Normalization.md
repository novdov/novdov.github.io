---
title: Batch Normalization
date: 2018-05-29 22:33:21
description: 딥러닝에서 정규화 방법으로 널리 사용되는 Batch Normalization (배치 정규화)에 대해 간단히 살펴봅니다.
categories:
- DeepLearning
tags:
- deep learning
---

> 참고: 밑바닥부터 시작하는 딥러닝 (사이토 고키 저, 개앞맵시 역, 한빛미디어)

배치 정규화는 2015년 [Sergey Ioffe, Christian Szegedy](https://arxiv.org/pdf/1502.03167.pdf)가 제안한 방법이다. 배치 정규화는 널리 사용되는 방법인데 그 이유는 다음과 같다.

- 학습을 빨리 진행할 수 있다.
- 초깃값에 크게 의존하지 않는다.
- 과대적합을 억제한다.

배치 정규화의 기본 아이디어는 각 층에서의 활성화값이 적당히 분포되도록 조정하는 것이다. 학습 시 미니배치 단위로 정규화하는데 구체적으로는 데이터 분포가 평균이 0, 분산이 1이 되도록 정규화한다.

$$\mu_B \leftarrow \dfrac{1}{m}\sum_{i=1}^m x_i $$

$$\sigma_B^2 \leftarrow \dfrac{1}{m}\sum_{i=1}^m (x_i - \mu_B)^2$$

$$\hat{x}_i \leftarrow \dfrac{x_i - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}}$$

여기에는 미니배치 $B = \{ x_1, x_2, \cdots, x_m \}$ 이라는 $m$ 개의 입력 데이터의 집합에 대해 평균 $\mu_B$ 와 분산 $\sigma_B^2$ 를 구한다. 그리고 입력 데이터를 평균이 0, 분산이 1인 데이터 $\{ \hat{x}_1, \hat{x}_2, \cdots, \hat{x}_m \}$ 가 되도록 정규화한다.

또 배치 정규화 계층마다 이 정규화된 데이터에 고유한 확대(scale)와 이동(shift) 변환을 수행한다.

$$y_i = \gamma \hat{x}_i + \beta$$

위 식에서 $\gamma$ 가 확대를, $\beta$ 가 이동을 담당한다. 두 값은 $\gamma = 1$, $\beta = 0$ 부터 시작한다. (1배 확대 및 0 이동 즉, 원본 그대로에서 시작)