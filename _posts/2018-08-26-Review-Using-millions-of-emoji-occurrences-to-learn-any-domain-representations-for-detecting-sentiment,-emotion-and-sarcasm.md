---
title: [Review] Using millions of emoji occurrences to learn any-domain representations for detecting sentiment, emotion and sarcasm
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

