---
title: '[Review] Searching for Activation Functions'
date: 2018-07-02 01:42:06
description: 활성화 함수는 딥러닝에서 중요한 역할을 담당합니다. ReLU를 대체할 활성화 함수, Swish를 강화학습을 통해 찾아낸 논문인 Searching for Activation Functions를 간단하게 리뷰합니다.
categories:
- DeepLearning
tags:
- deep learning
- review
---

> 출처: [Prajit Ramachandran, Barret Zoph, Quoc V. Le, Searching for Activation Functions, 2017](https://arxiv.org/pdf/1710.05941.pdf)

## Introductin

- ReLU는 그 간편함에 시그모이드나 tanh보다 더 널리, 딥러닝 커뮤니티에서 각광받는 활성화 함수가 되었음
- Maas et al., 2013; He et al., 2015; Clevert et al., 2015; Klambauer et al., 2017 등이 ReLU를 대체할 활성화 함수를 제안했지만 ReLU만큼 널리 채택되진 않았음. 많은 이들이 ReLU의 간편함과 안정성을 선호했는데 이는 다른 활성화 함수들은 다른 모델과 데이터셋에서 불안정한 모습을 보였기 때문임
- ReLU를 대체할 활성화 함수들은 사람의 손으로 설계되었음. 그러나 Zoph & Le, 2016; Bello et al., 2017; Zoph et al., 2017 등의 연구는 사람이 설계하던 부분을 자동화하는 검색 기술이 매우 효과적임을 보임. 특히 Zoph et al., 2017는 convolutional cell을 찾기 위해 강화학습 기반의 검색 기술을 사용했는데 ImageNet에서 사람이 설계한 아키텍처보다 더 높은 성능을 보임
- 본 논문에서는 새로운 활성화 함수를 찾기 위해 자동화 검색 기술을 사용했음. 특히 아키텍처를 변경하지 않으면서도 ReLU를 대체할 수 있는 스칼라 함수를 찾는데 중점을 두었음. (입력과 출력이 모두 스칼라) 철저한 검색과 강화학습 기반의 검색으로  유망한 여러 새로운 활성화 함수를 찾아냄. 
- 실증적 평가를 거쳐 찾아낸 함수는 Swish로, 기존 시그모이드에 x 대신 beta・x를 입력으로 주며, 그 시그모이드값에 x를 곱한 형태이다. 베타는 상수 혹은 훈련 가능한 파라미터이다. ReLU를 Swish로 교체한 결과 ImageNet top-1 분류에서 Mobile NASNet-A은 0.9%, Inception-ResNet-v2은 0.6% 만큼 성능이 향상되었음. 이는 꽤 대단한데 Inception V3 (2016)에서 Inception-ResNet-v2 (2017)로 1.3% 성능 향상을 위해 일 년간 파라미터 튜닝을 거쳤기 때문

## Methods

- 검색 공간을 설계하는 데 있어 그 크기와 표현성의 밸런스가 필요

- 활성화 함수는 여러 core unit(이항 함수)의 반복으로 구성되었음

  ![](https://i.imgur.com/fugNjYJ.png)

- 큰 검색 공간에는 RNN 컨트롤러를 사용함 (Zoph & Le, 2016). 각 타임 스텝마다 컨트롤러는 활성화 함수의 한 요소를 예측.

- RNN 컨트롤러는 검증 정확도를 최대화하기 위해 강화학습으로 훈련됨. 이는 RNN 컨트롤러가 가장 높은 검증 정확도를 내는 활성화 함수를 만들어내도록 유도

![](https://github.com/novdov/paper-read/blob/master/img/swish_02.png?raw=true)



## Search Findings

- 찾아진 활성화 함수들은 다음과 같음. 모든 함수는 ResNet-20을 child network로 사용하고 CIFAR-10 데이터에 대해 10K 스텝으로 실험되었음

![](https://github.com/novdov/paper-read/blob/master/img/swish_06.png?raw=true)

- 좋은 성능을 보인 활성화 함수들은 다음과 같음
  - 복잡한 함수는 간단한 함수보다 성능이 나쁨
  - 좋은 성능을 내는 함수들은 이항 함수에 $x$ 를 그대로 넣는 $b(x, g(x))$ 의 형태. ReLU의 $g(x)$ 는 $g(x)=0$
  - 나눗셈을 사용하는 함수들은 대체로 나쁜 성능을 보이는데 분모가 0에 가까우면 출력값이 폭주하기 때문. 나눗셈이 잘 동작할 때는 1) 분모가 0에서 멀거나, 분자 또한 0에 가까워 출력이 1이 될 경우

![](https://github.com/novdov/paper-read/blob/master/img/swish_03.png?raw=true)

- 더 견고한 함수를 찾아내기 위해 텐서플로에서 RestNet-164 (RN), Wide ResNet 28-10 (WRN), DenseNet 100-12 (DN) 세 모델을 대상으로 ReLU를 해당 함수로 대체하는 실험을 진행함

![](https://github.com/novdov/paper-read/blob/master/img/swish_04.png?raw=true)

- 모델의 변경에도 불구하고 8개 중 6개 모델이 성공적인 일반화 성능을 보임. $x \cdot \sigma (\beta x)$와 $\max(x, \sigma(x))$가 세 모델에서 모두 ReLU보다 높은 성능을 보임
- 더 좋은 일반화 성능을 보인 $x \cdot \sigma (\beta x)$, Swish 함수에 대해 ReLU와 비교하며 추가적인 평가를 진행함.



## Swish

![](https://github.com/novdov/paper-read/blob/master/img/swish_05.png?raw=true)

- Swish는 $\beta$ 가 0이면 스케일된 선형 ($x/2$) 이고 무한대로 갈수록 0-1 함수, 즉 ReLU에 가까워짐. $\beta$ 를 조절함으로써 선형과 ReLU 사이의 비선형 정도를 조절할 수 있음

- ReLU와 마찬가지로 Swish또한 상하한선이 있음. 반면 ReLU와는 달리 좀 더 부드러우며, 단조롭지 않음. 이 점이 Swish가 다른 활성화 함수와 구별되는 점

- ReLU와 Swish의 가장 큰 차이점인 입력값 $x$ 가 음수일 경우의 "bump"임.

- 더군다나 대부분의 딥러닝 라이브러리에서 Swish는 코드 한줄로 바로 실행할 수 있음 (`x * tf.sigmoid(beta * x) `  혹은 `tf.nn.swish(x)`). 또한 ReLU 사용시보다 learning rate를 약간 낮추는 편이 잘 동작했음

  

## Experiments with Swish

- ResNet-v2나 Transformer 등의 모델로 Swish와 여러 활성화 함수를 비교함

![](https://i.imgur.com/mfbqCwv.png)



## Conclusion

- Swish는 간단하며 ReLU와 비슷한데, 이는 네트워크에서 ReLU를 교체하는 것은 코드 한 줄이면 된다는 것을 의미