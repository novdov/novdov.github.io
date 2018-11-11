---
title: '[Review] You May Not Need Attention'
date: 2018-011-06 09:23:00
description: 최근 활발히 사용되고 있는 Attention을 적용하지 않은 NMT 모델을 소개한 논문을 리뷰합니다.
categories:
- NLP
- Deep Learning
tags:
- nlp
- deep learning
- review
---



## Introduction

최근 활발히 연구되고 있는 NMT 모델은 다음의 속성을 따른다.

- Decoder는 source sequence representation에 대해 attention을 사용한다.
- Encoder와 Decoder는 두 개의 다른 모듈이며, Encoder는 Decoder의 작동 전에 source sentence의 encoding을 마쳐야 한다.

논문에서는 NMT 모델이 위의 속성 없이 얼마나 성능을 낼 수 있는지 확인한다. 논문은 기존의 Encoder-Decoder with attention 모델과 다른 구조로 NMT를 실험했고, 이 모델을 **eager translation model**이라고 부른다.

이를 위해 논문에서는 Bahdanau et al. (2014)의 모델로 시작해 attention을 제거하고 Encoder와 Decoder를 하나의 간단한 모델로 합친다. 이 모델은 Zaremba et al. (2014)의 langaugae model을 닮았다. 이를 통해 논문의 모델은 source word가 입력되지마자 번역을 시작할 수 있다. Eager translation model은 일정 개수의 메모리만 사용하는데 이는 매 타임 스텝마다 직전 타임 스텝의 히든 스테이트만 이용하기 때문이다. 한편, 논문의 모델이 다른 모델과 가장 큰 차이점을 보이는 곳은 preprocessing이다.



## Data Preprocessing

논문에서 소개하는 preprocessing의 핵심은 source/target 문장의 **align** (정렬)이다. 논문에서는 이를 위해 source/target sentence에 특정 속성을 요구한다.

- source/target 문장의 대응 관계를 inferring 하는데, 이 때 각 target 단어는 최대 하나의 source 단어에 매칭된다. 그리고 정렬된 $(s_i, \boldsymbol{t}_j)$ 쌍이 $i \le j$ 를 만족하는 상태를 *eager feasible* 하다고 부른다.

논문에서는 위의 속성을 만족시키기 위해 먼저 off-the-shelf 모델 (`fast_align`; [Dyer at el., 2013](http://www.aclweb.org/anthology/N13-1073)) 을 사용해 source/target 문장 간의 대응을 예측한 뒤 최소한의 $\epsilon$ (empty) 토큰을 target 문장에 삽입한다. 이 $\epsilon$ 토큰은 학습과 예측 단계에서만 사용되고 실제 번역 문장을 만들어낼 때에는 제거된다.

> `fast_align` 모델은 source/target 문장 간 대응의 maximum likelihood를 log-linear 속도로 찾아내는 간단하고 빠른 모델이다.

예를 들어 source 문장 “El perro blanco” (“The dog white”)의 올바른 번역은 “The white dog.”이다. 영어 문장의 두 번째 단어 (white)와 스페인 문장의 세 번째 단어가 분명하게 대응 (정렬)된다는 것을 가정하면 시퀀스를 eager feasible하게 만들기 위해 논문의 모델은 target 문장을 “The $\epsilon$ white dog.” 으로 변환한다.

논문에서 소개하는 eager feasible 알고리즘은 아래와 같다.

- source 문장을 $s = \langle s_1, \ldots, s_m \rangle$ , target 문장을 $\boldsymbol{t} = \langle t_1, \ldots, t_n \rangle$ 이라고 했을 때 $\mathcal{A}$ 를 정렬 쌍 $(i, j)$ 의 집합으로 둔다. ($t_j$ 가 $s_i$ 에 대응된다.)
- target 문장을 왼쪽에서 오른쪽으로 살펴본다. 현재 target 단어가 $t$ 이고 현재의 위치는 $j$ 로 가정한다. $t$ 가 source 단어 $s_i$ 와 정렬되고 (에 대응하고) $i \le j$ 이면 다음 target 단어로 넘어간다. (정렬된 상태이므로)
- 그렇지 않을 경우 해당 target 단어 바로 앞에 $\epsilon$ 토큰을 삽입한다. 토큰 삽입으로 $t$ 는 target 문장에서 $i$ 위치로 이동한다. 물론, 다른 target 단어들도 오른쪽으로 이동한다.

- 또한 위의 작업을 조금 더 간단하게 하기 위해 논문에서는 모든 target 문장 앞에 $b \in \{ 0, 1, \ldots, 5 \}$ 개의 $\epsilon$ 토큰을 미리 더한 뒤 실험했다. 이는 모델이 번역 전에 더 많은 source 문장을 소비하도록 한다. 즉 예측 시에 모델이 $b$ 개의 $\epsilon$ 을 번역 전에 만들어내도록 강제한다. 논문의 모델은 문장 앞의 $\epsilon$ 도 고려함으로써 target 문장 사이에 삽입해야 할 $\epsilon$ 의 개수를 줄인다.
- 위의 과정 후에 source/target 문장의 길이가 동일하지 않을 경우 짧은 문장 뒤에 $\epsilon$ 을 추가해 두 문장의 길이를 맞춘다.



## Model

- 매 타임 스텝마다 모델은 source 언어의 현재 입력 단어와 target 언어의 출력 단어를 각각 $E$ 차원으로 임베딩한다.   이 두 벡터는 $2E$ 차원으로 concat 된 뒤, multi-layer LSTM에 입력된다. LSTM의 출력은 FC layer를 이용해 $E$ 차원으로 변환된다. 이 벡터는 출력 임베딩과 softmax 를 이용해 target 사전의 distribution으로 변환된다. 모델은 또한 source 언어의 입력 임베딩, target 언어의 입력 임베딩, 출력 임베딩을 공유한다.
- 논문에서 소개하는 모델은 Zaremba et al. (2014)의 recurrent language model과 매우 닮아 있다.



  