---
title: '[Review] Universal Trnasformers'
date: 2018-09-23 09:23:00
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

## Introduction

- Transformer 모델 같은 convolutional & fully-attentional feed-forward 아키텍처는 최근 MT와 같은 시퀀스 모델링에서 RNN의 대체제로 떠올랐다. Transformer 모델은 self-attention 메커니즘을 통해 입출력 심볼의 context-informed vector-space representation을 학습한다. 이 representation은 모델이 symbol-by-symol로 출력 시퀀스를 예측하기 때문에, 서브시퀀트 심볼의 분산을 예측한다. Transformer 모델의 이러한 점은 병렬화하기 쉽고, 각 심볼의 representaion이 다른 심볼의 representaion에 의해 직접적인 정보를 담기 때문에, 효과적인 receptive field를 얻을 수 있다는 이점을 가져온다.

- Transformer 모델은 반복적이고 재귀적인 transformation을 학습하는 RNN의 inductive bias에 앞서지만, 이 inductive bias가 언어의 복잡성을 학습하는데 중요한 역할을 한다는 것을 실험 결과가 보여준다. 이 때문에 transformer는 계산적으로 일반화하기 매우 어렵다. Universal Transformer는 이러한 단점들을 극복한다.

![](https://i.imgur.com/Fc9cipx.png)



##  Model

### The Universal Transformer

![](https://i.imgur.com/zAP064b.png)

- Universal Transformer는 seq2seq 모델에서 흔히 쓰이는 encoder-decoder 구조에 기반한다.  그러나 Universal Transformer는 시퀀스 위치를 순환하지 않고, 각 위치의 vector representaion의 연속적인 갱신(revision)을 순환한다는 점에서 기존의 RNN과 가장 큰 차이를 가진다. **즉 Universal Transformer는 시퀀스 내의 심볼 개수에 구애받지 않고, 각 심볼의 representaion의 업데이트 횟수에 귀속된다.**

- 매 스텝마다 각 위치에서의 representation은 2단계에 걸쳐 갱신된다.
  - 먼저, Universal Transformer는 self-attention 메커니즘을 사용해 모든 위치에서 정보를 교환하고 각 위치의 representation을 생성한다. 이 representation은 이전 타임 스텝의 representation에 영향을 받는다.
  - 그 다음, 각 위치에서 독립적으로 self-attention 출력값에 *shared* transition을 적용한다. 이 점이 레이어를 쌓는 Transformer나 RNN 등 유명한 neural 시퀀스 모델과 가장 차별되는 점이다.
- Encoder로는 $m$ 길이의 입력이 주어졌을 때, $d$ 차원의 임베딩으로 초기화된 행렬을 이용한다. ($H^0 \in \mathbb{R}^{m \times d}$) Universal Transformer는 그 다음 반복해서 $t$ 스텝에서의 $m$ 위치의 represantation $H^t$을 계산하는데, 이 때 multiheaded dot-product self-attention, recurrent transition을 적용한다. residual connection과, dropout, layer normalization 또한 함께 적용된다.
  - 작업에 따라 transition은 separable convolution이나 fully-connected nn (with relu) 중 하나가 사용된다.
- $T$ 스텝 이후에 Universal Transformer의 최종 출력값은 입력 시퀀스의 $m$ 심볼의 $d$ 차원 representation 행렬이다. ($H^T \in \mathbb{R}^{m \times d}$)
- Decoder는 기본적으로 encoder와 동일한 구조를 가진다. 하지만, decoder는 self-attention 이후에 decoder represention에서 얻은 쿼리 $Q$ 와 encoder representation을 projection해서 얻은 key/value $K, V$ 를 이용해 입력 시퀀스 각 위치의 최종 encoder representation $H^T$ 으로 향하는 attention을 추가로 계산한다.
- 학습 동안 Decoder  입력은 encoder-decoder 구조와 동일하게 오른쪽으로 하나의 위치만큼 이동한 목표 출력이다.
- 마지막으로 목표 심볼 distribution은 최종 decoder state에서 출력 사전 크기 $V$ 로 affine 변환 $O \in \mathbb{R}^{d \times V}$ 과 softmax를 통해 얻어진다.

$$
p(y_{pos} \vert y_{[1:pos-1]}, H^T) = \text{softmax}(OH^T)^1
$$



### The Adaptive Universal Transformer

