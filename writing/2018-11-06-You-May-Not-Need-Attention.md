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

논문에서는 NMT 모델이 위의 속성 없이 얼마나 성능을 낼 수 있는지 확인한다.

이를 위해 논문에서는 Bah- danau et al. (2014)의 모델로 시작해 attention을 제거하고 Encoder와 Decoder를 하나의 간단한 모델로 합친다. 이 모델은 Zaremba et al. (2014)의 langaugae model을 닮았다. 이를 통해 논문의 모델은 source word가 입력되지마자 번역을 시작할 수 있다. Eager translation model은 일정 개수의 메모리만 사용하는데 이는 매 타임 스텝마다 직전 타임 스텝의 히든 스테이트만 이용하기 때문이다. 한편, 논문의 모델이 다른 모델과 가장 큰 차이점을 보이는 곳은 preprocessing이다.



## Data Preprocessing

논문에서 소개하는 preprocessing의 핵심은 **align**이다. 논문에서는 이를 위해 source/target sentence에 특정 속성을 요구한다.

