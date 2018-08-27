---
title: '[Review] Using millions of emoji occurrences to learn any-domain representations for detecting sentiment, emotion and sarcasm'
description: 이모지를 이용한 domain representation 학습과 레이어의 각 부분을 따로 학습 시키는 transfer learning "chain-thaw”를 소개한 논문을 리뷰합니다.
date: 2018-08-26
category:
- DeepLearning
- NLP
tags:
- nlp
- deep learning
---

> Bjarke Felbo, Alan Mislove, Anders Søgaard, Iyad Rahwan, Sune Lehmann. 2017. Using millions of emoji occurrences to learn any-domain representations for detecting sentiment, emotion and sarcasm. ACL 2017, pages 1615–1625

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



