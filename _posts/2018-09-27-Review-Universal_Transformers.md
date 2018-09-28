---
title: '[Review] Universal Transformers'
date: 2018-09-27 09:23:00
description: 계산적으로 일반화하기 어렵고, RNN에 비해 간단한 태스크도 잘 처리 못했던 기존의 Trnaformer 모델을 개선한 Universal Transformers 모델을 소개한 논문을 리뷰합니다.
categories:
- NLP
- Deep Learning
tags:
- nlp
- deep learning
- review
---

> [Mostafa Dehghani et al., Universal Transformers., 2018](https://arxiv.org/abs/1807.03819)
>
> 해당 코드: https://github.com/tensorflow/tensor2tensor

## Introduction

- Transformer 모델 같은 convolutional & fully-attentional feed-forward 아키텍처는 최근 MT와 같은 시퀀스 모델링에서 RNN의 대체제로 떠올랐다. Transformer 모델은 self-attention 메커니즘을 통해 입출력 심볼의 context-informed vector-space representation을 학습한다. 이 representation은 모델이 symbol-by-symol로 출력 시퀀스를 예측하기 때문에, 서브시퀀트 심볼의 분산을 예측한다. Transformer 모델의 이러한 점은 병렬화하기 쉽고, 각 심볼의 representaion이 다른 심볼의 representaion에 의해 직접적인 정보를 담기 때문에, 효과적인 receptive field를 얻을 수 있다는 이점을 가져온다.
- Transformer 모델은 반복적이고 재귀적인 transformation을 학습하는 RNN의 inductive bias에 앞서지만, 이 inductive bias가 언어의 복잡성을 학습하는데 중요한 역할을 한다는 것을 실험 결과가 보여준다. 이 때문에 transformer는 계산적으로 일반화하기 매우 어렵다. Universal Transformer는 이러한 단점들을 극복한다.

![](https://i.imgur.com/Fc9cipx.png)


##  Model

### The Universal Transformer

![](https://i.imgur.com/zAP064b.png)

- Universal Transformer는 sequence-to-sequence 모델에서 흔히 쓰이는 encoder-decoder 구조에 기반한다.  그러나 Universal Transformer는 시퀀스 위치를 순환하지 않고, 각 위치의 vector representaion의 연속적인 갱신(revision)을 순환한다는 점에서 기존의 RNN과 가장 큰 차이를 가진다. **즉 Universal Transformer는 시퀀스 내의 심볼 개수에 구애받지 않고, 각 심볼의 representaion의 업데이트 횟수에 귀속된다.**
- 매 스텝마다 각 위치에서의 representation은 2단계에 걸쳐 갱신된다.
  - 먼저, Universal Transformer는 self-attention 메커니즘을 사용해 모든 위치에서 정보를 교환하고 각 위치의 representation을 생성한다. 이 representation은 이전 타임 스텝의 representation에 영향을 받는다.
  - 그 다음, 각 위치에서 독립적으로 self-attention 출력값에 *shared* transition을 적용한다. 이 점이 레이어를 쌓는 Transformer나 RNN 등 유명한 neural 시퀀스 모델과 가장 차별되는 점이다.

  $$
  \text{Attention}(Q, K, V) = \text{softmax}(\dfrac{QK^T}{\sqrt(d)})V \\
  \text{MultiHeadSelfAttention}(H) = \text{Concat}(\text{head}_1, \ldots, \text{head}_k)W^O \\
  \text{where head}_i = \text{Attention}(HW_i^Q, HW_i^K, HW_i^V)
  $$

- Encoder로는 $m$ 길이의 입력이 주어졌을 때, $d$ 차원의 임베딩으로 초기화된 행렬을 이용한다. ($H^0 \in \mathbb{R}^{m \times d}$) Universal Transformer는 그 다음 반복해서 $t$ 스텝에서의 $m$ 위치의 represantation $H^t$을 계산하는데, 이 때 multiheaded dot-product self-attention, recurrent transition을 적용한다. residual connection과 dropout, layer normalization 또한 함께 적용된다.

  - 작업에 따라 transition은 separable convolution이나 fully-connected NN (with relu) 중 하나가 사용된다.
- $T$ 스텝 이후에 Universal Transformer의 최종 출력값은 입력 시퀀스의 $m$ 심볼의 $d$ 차원 representation 행렬이다. ($H^T \in \mathbb{R}^{m \times d}$)
- Decoder는 기본적으로 encoder와 동일한 구조를 가진다. 하지만, decoder는 self-attention 이후에 decoder represention에서 얻은 쿼리 $Q$ 와 encoder representation을 projection해서 얻은 key/value $K, V$ 를 이용해 입력 시퀀스 각 위치의 최종 encoder representation $H^T$ 으로 향하는 attention을 추가로 계산한다.
- 학습 동안 Decoder  입력은 encoder-decoder 구조와 동일하게 오른쪽으로 하나의 위치만큼 이동한 목표 출력이다.
- 마지막으로 목표 심볼 distribution은 최종 decoder state에서 출력 사전 크기 $V$ 로 affine 변환 $O \in \mathbb{R}^{d \times V}$ 과 softmax를 통해 얻어진다.

$$
p(y_{pos} \vert y_{[1:pos-1]}, H^T) = \text{softmax}(OH^T)^1
$$

### The Adaptive Universal Transformer

- 시퀀스 프로세싱 중에, 특정 심볼들은 다른 것들보다 모호할 때가 있어 이 심볼들을 처리하는 데 자원을 더 쏟는 것이 필요하다. 이 때 각 심볼에 필요한 계산량을 조절하는 ACT (Adaptive Computation Time)을 Universal Transformer에 적용한 것을  The Adaptive Universal Transformer라고 부른다.


## Experiments

- Universal Transformer의 실험은 총 6가지 태스크를 통해 이뤄졌다. 각각은 다음과 같다. 
  - bAbI Question-Answering
  - Subject-Verb Agreement
  - LAMBADA Language Modeling
  - Algorithmic Tasks
  - Learning to Execute (LTE)
  - Machine Translation

### bAbI Question-Answering

- bAbI Question-Answering 데이터셋은 주어진 영어 문장에서 supporting facts를 인코딩하는 질문에 답하는 20가지의 태스크로 이루어져 있다.
- Adaptive Universal Transformer가 10K/1K 모두에서 SOTA 성능을 보였다.

![](https://i.imgur.com/9vMSNMB.png)


### Subject-Verb Agreement

- subjec와 verb 일치를 평가하는 태스크로, hierarchical (dependency) 구조를 얼마나 잘 잡아내는 지 확인하는 태스크이다.
- Universal Transformer는 기존의 Transformer보다 나은 성능을 보였고, Adaptive Universal Transformer는 SOTA와 견줄만한 성능을 보였다.

![](https://i.imgur.com/UivpQHy.png)


### LAMBADA Language Modeling

- LAMBADA Language Modeling 태스크는 주어진 문장을 통해 공백 단어를 예측하는 문제이다.
- 해당 태스크에서도 Universal Transformer가 SOTA 성능을 이뤄냈다.

![](https://i.imgur.com/3uH0c6I.png)


### Algorithmic Tasks

- Universal Transformer가 LSTM과 Transformer보다 나은 성능을 보였다.
- Neural GPU가 완벽한 결과를 보이지만, 이는 특별한 과정이 추가되었기 때문이며, 다른 모델은 그렇지 않다.

![](https://i.imgur.com/nz9QN9a.png)


### Learning to Execute (LTE)

- 이 태스크는 컴퓨터 프로그램을 실행하도록 학습할 수 있는지 평가하는 문제이다.
- Universal Transformer는 모든 태스크에서 완벽한 결과를 보였다.
  - 위: char-acc (maximum length of 55)
  - 아래: char-acc (maximum nesting of 2 nad length of 5)

![](https://i.imgur.com/NYneV7H.png)

![](https://i.imgur.com/loFRg0Y.png)


### Machine Translation

- MT 태스크는 WMT 2014 영어-독일어로 평가되었다.
- Universal Transformer (with fully-connected recurrent function without ACT) 가 기본 Transformer보다 BLEU를 0.9 향상시켰다.

![](https://i.imgur.com/NH8fGw1.png)


## Universality and Relationship to Other Models

- Universal Transformer는 end-to-end Memory Network와도 관련되어 있다. 그러나 end-to-end Memory Network와는 다르게 Universal Transformer는 개별 입/출력 위치에 정렬된 스테이트에 해당하는 메모리를 사용한다. 또한, Universal Transformer는 encoder-decoder 구조를 따르며 대규모 sequence-to-sequence 태스크에서 좋은 성능을 보인다.


## Conclusion

- Universal Transformer는 다음의 주요 속성을 하나의 모델에 결합했다.
  - Weight sharing
    - CNN/RNN에서 사용되는 weight sharing을 도입해 소규모/대규모 태스크에서 inductive bias와 모델의 표현력 사이에서 경쟁력 있는 능력을 갖췄다.
  - Conditional Computation
    - Universal Transformer는 ACT를 도입해 fixed-depth Universal Transformer보다 더 강력한 능력을 갖췄다. (The Adaptive Universal Transformer)


## Appendix

### Detailed Schema of the Universal Transformer

![](https://i.imgur.com/aX52RnY.png)
