---
title: '[Review] Using millions of emoji occurrences to learn any-domain representations for detecting sentiment, emotion and sarcasm'
description: 이모지를 이용한 domain representation 학습과 레이어의 각 부분을 따로 학습 시키는 transfer learning 방법 "chain-thaw”를 소개한 논문을 리뷰합니다.
date: 2018-08-26
category:
- DeepLearning
- NLP
tags:
- nlp
- deep learning
---

> [Bjarke Felbo, Alan Mislove, Anders Søgaard, Iyad Rahwan, Sune Lehmann. 2017. Using millions of emoji occurrences to learn any-domain representations for detecting sentiment, emotion and sarcasm. ACL 2017, pages 1615–1625](https://arxiv.org/abs/1708.00524)

## Introduction

- 레이블링 된 데이터의 부족으로 NLP 작업이 제한되는 경우가 있다. 이런 이유로 문장에 나타나는 감정 표현은 소셜 미디어 감성 분석과 연관 작업에서 representation을 학습하는 데에 유용하게 사용되고 있다.
- 이를 이용한 State-of-the-art 접근법으로는 소셜 미디어 감성 분석에 긍정/부정 이모티콘을 사용했고 (Deriu et al., 2016; Tang et al., 2014), 이와 비슷하게 #anger, #joy 같은 해시태그를 사용해 감정 분석에 접근한 방법도 있다. (Mohammad, 2012)
- 노이즈가 섞인 레이블을 이용한 Distant supervision은 많은 경우에 모델의 성능을 향상시킨다. 본 논문에서는 더 다양한 노이즈가 섞인 레이블로 Distant supervision을 확장하고, 이러한 시도가 텍스트에서 더 풍부한 감정 representation 학습을 가능케 했다. (DeepMoji) 본 논문에서는 단일 pretrained model을 5가지 도메인으로 일반화한다.


## Method

### Pretraining

- 논문은 많은 경우에 이모지가 텍스트의 감정 정보를 시사하는 사실을 바탕으로, 이모지를 예측하는 분류 모델을 미리 학습하는 것이 목표 작업의 성능을 향상시키다고 말한다.

- 데이터로는 2013.01.01~ 2017.06.01 간의 트윗을 사용했으나, 이모지가 포함된 텍스트라면 상관 없다. 한편, URL이 포함된 트윗은 해당 URL이 감정 표현의 대상이 된다고 가정하고 URL이 없는 트윗만 사용되었다.
- 토크나이징은 단어 기준으로 진행되었고, 중복된 토큰은 정규화되었다. (‘loool’ = ‘looooool’) URL (벤치마크 데이터셋)과 숫자도 마찬가지로 정규화되었다.
- 이모지에 대해서는, 사용된 이모지의 개수와 상관없이 같은 종류의 이모지면 pretraining을 위한 데이터로 사용되었다.

### Model

- DeepMoji 모델 구성은 다음과 같다.
  - Embedding (256 dim) & tanh (L2 regularization of 1E-6)
  - 2 bi-LSTM (1024 dim, 512 dim each)
  - Attention layer

![](https://i.imgur.com/112C7M6.png?1)

### Transfer Learning

- pretrain 된 모델은 transfer learning을 통해 목적 작업에 적용될 수 있는데, 본 논문에서는 “chain-thaw”라는 간단한 접근법을 소개한다. “chain-thaw” 방법은 순차적으로 가중치를 고정하고, 한번에 하나의 레이어만 업데이트하는 방법이다.
- 구체적으로는 먼저 어느 한 레이어를 학습시키고, (보통 Softmax 레이어) 그 다음 첫 번째 레이어부터 순차적으로 업데이트한다. 마지막에는 모든 레이어가 업데이트된다.
- chain-thaw을 통해 오버피팅의 리스크를 줄이면서 어휘를 새로운 도메인으로 확장할 수 있다.

![](https://i.imgur.com/jZfN6DA.png?1)



## Experiments

### Emoji prediction

- pretraining 데이터에서 트윗은 하나의 이모지에 대응하도록 복사되었고, 이렇게 준비된 데이터는 트윗 16억 개이다. validation/test 데이터로는 각각 640K 개 (각 이모지 당 10K 개)가 사용되었다. 나머지 트윗은 모두 upsampling을 거쳐 학습 데이터로 사용되었다.
- DeepMoji 모델은 pretraining task로 평가되었고, 결과는 아래의 표와 같다. top 1과 top 5 정확도 모두 노이즈가 섞인 이모지 레이블로 평가되었다. (어떤 문장에도 적합한 이모지 등)

|                     | Params | Top 1 | Top 5 |
| ------------------- | :----: | :---: | :---: |
| Random              |        | 1.6%  | 7.8%  |
| fasttext (d = 256)  |  12.8  | 12.8% | 36.2% |
| DeepMoji (d = 512)  |  15.5  | 16.7% | 43.3% |
| DeepMoji (d = 1024) |  22.4  | 17.0% | 43.8% |

- fastText만 사용한 모델은 DeepMoji의 임베딩 레이어만 사용한 것과 동일한 결과를 냈다. 한편, fastText와 DeepMoji의 Top 5 정확도 차이는 이모지 예측의 어려움을 보여준다.
- DeepMoji는 임베딩과 Softmax 레이어 사이에 어텐션 레이어가 있는 LSTM 레이어를 가지는데, 이 차이가 각 단어의 context를 잡아내는 데 중요한 역할을 했다고 말해준다.
- 요즘의 모델들은 기본적으로 biLSTM과 어텐션 레이어 등을 포함하는데, (biLSTM은 거의 기본값이 된 듯 한다.) 이것은 그만큼 biLSTM과 어텐션의 성능이 NLP 작업에서 뛰어나다는 것을 보여준다고 생각한다.


## Benchmarking

- 논문에서는 5개 도메인의 8개 데이터를 이용해 3개의 NLP 작업을 수행해 모델의 벤치마크 성능을 평가했다.
- 평가 metric으로는 감정 분석과 sarcasm에는 F1이, 감성 분류에는 정확도가 사용되었다.

![](https://i.imgur.com/EtJLLUw.png)

- 벤치마킹 결과는 chain-thaw 방법을 활용한 DeepMoji 모델은 state-of-the-art 모델보다 모든 데이터셋에서 더 높은 성능을 보였다. 벤치마킹 결과는 DeepMoji의 좋은 성능에는 chain-thaw 영향이 크다는 것을 말해준다.


## Model Analysis

- DeepMoji와 이전의 distant supervison 방법들의 가장 큰 차이는 DeepMoji는 노이즈가 섞인 레이블을 다양하게 활용했다는 점이다. 다양한 이모지의 영향을 알아보기 위해 1/8로 줄인 이모지 데이터셋 (긍/부정)을 활용한 결과, 벤치마크 상의 성능은 데이터의 크기보다 레이블의 다양성과 더 연관이 있었다.
- 이모지는 비슷한 감정을 나타내지만, 문맥마다 미묘한 차이를 나타낸다. DeepMoji는 이러한 미묘함을 학습했고, 이는 성능 향상으로 이어졌다는 것이다.

- transfer learning 성능 향상에 대해 논문은 이렇게 추측한다. 1) skip connections를 적용한 어텐션 메커니즘 덕분에 모델이 어느 time step에서도 low-level features에 쉽게 접근해서 새로운 작업에 이용할 수 있다는 점, 2) 작은 데이터셋으로의 transfer learning에서 skip connections 덕분에 출력 레이어에서 초기 레이어로의 gradient 흐름이 개선된 점


## Conclusion

- 이 논문은 이모지를 이용해 감정 정보를 학습했고, 개선된 transfer learning으로 NLP 작업에서의 성능을 개선했다. DeepMoji 모델 또한 감정이라는 context를 학습한 contextualized embedding이라고도 볼 수 있을 것 같다.
- 또한 살펴볼 것은, 2-layer BiLSTM과 어텐션 메커니즘, skip connections의 사용이다. 최근 소개되는 state-of-the-art 모델들은 거의 대부분 어텐션 메커니즘이 추가된 BiLSTM을 이용하고 있다. 그만큼 NLP 작업에서 BiLSTM과 어텐션 메커니즘의 성능이 뛰어나다는 것일 것이다.
- 다만, 이러한 큰 네트워크를 학습하기 위해서는 그만큼 풍부한 데이터도 중요하다고 생각한다. State-of-the-art contextualized embedding인 ELMo의 경우에도 4096 차원의 아주 큰 BiLSTM과 1B token dataset을 이용한 CNN을 이용한다. 결국 context 학습이 최종 목적이 아니라, 이 모델을 다르 작업에 transfer learning으로 적용하자고 하는 것이기 때문에 더더욱 그럴 것이라고 생각한다. CV 분야에서도 미리 학습된 VGG 등을 이용해 transfer learning을 적용하는 것과 궤를 같이 한다고 생각한다.
- 그러나 여기서 한국어 NLP에 적용하기 위한 한계점도 있는데, 영어권 자료에 비해 대규모 한국어 데이터셋은 공개된 것이 많이 부족하다는 것이다. 세종 코퍼스는 2000년대 초반에 종료된 프로젝트이며, 그 양도 많지 않아 아쉬울 뿐이다.
