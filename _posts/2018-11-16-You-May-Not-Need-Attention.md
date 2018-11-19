---
title: '[Review] You May Not Need Attention'
date: 2018-11-16 09:23:00
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
- 논문에서 소개하는 모델은 Zaremba et al. (2014)의 recurrent language model과 매우 닮아 있다. 모델에서처럼, 논문에서도 teacher forcing과 cross-entropy loss를 사용한다. padding 심볼도 target sentence의 한 부분으로 다뤄졌고, 이를 위한 별도의 loss나 objective funtion은 사용되지 않았다.
- 또한 다른 번역 모델과 달리 논문의 모델은 inference 중에 일정 양의 메모리만 사용한다. 메모리에 직전 히든 스테이트만 저장하고 인코딩된 이전 단어들의 representation은 저장하지 않는다. 이를 통해 decoding complexity는 최악의 경우 $\mathscr{O}(n+m)$ 이다.



### Aligned Batching

preprocessing 이후에 모든 source-target 쌍은 동일한 길이로 변한다. 논문에서는 이 다음 source 문장과 target 문장을 각각 순서를 유지한 string으로 합친다. 이를 통해 모델은 마치 language model을 학습하는 것처럼 학습할 수 있다. 구체적으로는 BPTT 하이퍼파라미터가 지정된다. 각 배치의 모든 요소는 BPTT source 토큰과 각각에 대응하는 target 토큰을 포함한다. language model을 학습할 때처럼, ($i-1$) 번째 배치의 마지막 히든 스테이트가 $i$ 번재 배치의 첫 히든 스테이트가 된다.



### Decoding

논문에서는 예측 (inference) 중에 출력의 질을 향상시키기 위해 beam search를 조정해 사용한다. 구체적인 조정은 다음과 같다.

- Padding limit: 논문에서는 $\epsilon$ 의 개수가 제한 개수에 다다르면 $\epsilon$ 의 등장 확률을 0으로 강제함으로써 패딩 토큰의 최대 개수에 제한을 두었다. 처음에 삽입되는 패딩 토큰은 이 제한 개수에 포함되지 않는다.
- Source padding injection (SPI): 논문에서는 decoder가 source 언어의 EOS 토큰을 읽으면 이 EOS 토큰에 높은 확률을 부여한다는 것을 발견했다. 이에 논문에서는 SPI 파라미터를 $c$ 로 두었고, beam search에서는 0부터 $c$ 까지는 EOS 전까지를 $\epsilon$ 으로 간주한다. 



## Experiments

### Setup

- 실험은 EN ⟷ FR, EN ⟷ DE 로 이루어졌고, 학습/검증/테스트 데이터는 각각 WMT 2014, newstest2013, newstest2014를 이용했다. 각 문장은 토크나이징과 30,000 BPE opeartion을 거친다.
- 네트워크의 구조는 먼저 4개의 LSTM 레이어 (1,000 units, embedding 500dim)를 이용한다. 학습 중 레이어와 임베딩에는 droutput이 적용된다. 배치 사이즈는 200, unroll step은 60이다. SGD를 사용했고, 첫 learning rate는 20이다. 논문에서는 6,500 스텝마다 검증 데이터에 대한 perplexity를 확인하고, 개선이 없으면 learning rate를 1/2로 줄인다.



### Results

![](https://i.imgur.com/f4HmYkX.png)

- eager 모델은 EN ⟷ FR에 대해서는 레퍼런스 모델보다 최대 0.8% 낮은 BLEU를 보였다. 그러나 EN ⟷ DE 에서는 최대 4.8% 레퍼런스보다 낮은 성능을 보였다.

  ![Imgur](https://i.imgur.com/k5O8rvO.png)

- 문장 길이로 모델의 성능을 확인했을 때는 긴 문장에서는 eager 모델이 더 좋은 성능을 보였으나 짧은 문장에 대해서는 레퍼런스 모델보다 낮은 성능을 보였다. 이는 attention 기반의 방법이 짧은 문장 학습에 어려움을 보이기 떄문이다.



## Conclusion

- 논문에서 소개한 방법은 토크나이징 결과와, source/target 문장 간의 어순, 닮음새에 영향을 많이 받을 것으로 보인다. 특히 한국어/영어 같이 문장의 서술 구조의 차이가 큰 경우에는 align이 쉽지 않을 뿐더러, 높은 성능을 기대하기도 어려울 것 같다.
- 다만 기존 NMT의 Encoder-Decoder 구조를 이용하지 않더라도 기존과 비슷한 성능을 보인다는 것은 모델의 구조 뿐만 아니라, 데이터 전처리 방법이 얼마나 중요한지 보여주는 대목이라고 생각된다.