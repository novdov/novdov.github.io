---
title: Corpus-based Chatbots
date: 2018-05-22 20:37:14
description: 말뭉치 기반 챗봇은 규칙을 이용하는 대신 인간 간 대화 혹은 인간-기계 간 대화에서의 인간의 반응을 학습한다. 보통 이용되는 말뭉치로는 Twitter, Weibo, 영화 대사 등이 있다. 말뭉치 기반 챗봇 시스템에는 두 타입이 있는데 하나는 정보 검색 기반(information retrieval based)이고 다른 하나는 시퀀스 변환 기반의 지도 학습 시스템이다.
categories:
- NLP
tags:
- deep learning
- nlp
---

> 이 글은 stanford의 Dan Jurafsky and James H. Martin가 쓴 Speech and Language Processing (3rd ed. draft) 중 ch 29.[Dialog Systems and Chatbots](https://web.stanford.edu/~jurafsky/slp3/29.pdf)을 참고했습니다.



## Corpus-based chatbots

말뭉치 기반 챗봇은 규칙을 이용하는 대신 인간 간 대화 혹은 인간-기계 간 대화에서의 인간의 반응을 학습한다. 보통 이용되는 말뭉치로는 Twitter, Weibo, 영화 대사 등이 있다.

말뭉치 기반 챗봇 시스템에는 두 타입이 있는데 하나는 정보 검색 기반(information retrieval based)이고 다른 하나는 시퀀스 변환 기반의 지도 학습 시스템이다.



### IR-based chatbots

IR 기반 챗봇의 원리는 사용자의 반응 $X$ 에 대해 말뭉치로부터 적절한 $Y$를 선택해 대응하는 것이다. 이는 말뭉치의 선택과 말뭉치로부터 어떤 것을 선택할지에 달려 있다. IR 기반 챗봇은 두 가지의 간단한 방법을 통해 말뭉치에서 적절한 답변을 선택한다.

#### 1) 사용자의 턴과 가장 유사한 턴에 대답 리턴

사용자 질의 $q$ 와 대화 말뭉치 $C$ 에 대해 $q$ 와 가장 유사한(코사인 유사도 등) $t$ 턴을 찾고 그에 이어지는 대답을 리턴한다.

$$r = response \Bigl( \underset{t \in C}{\text{argmax}}\dfrac{q^Tt}{\| q \| t \|} \Bigr)$$



#### 2) 가장 유사한 턴 리턴

사용자 질의 $q​$ 와 대화 말뭉치 $C​$ 에 대해 $q​$ 와 가장 유사한(코사인 유사도 등) $t​$ 를 리턴한다. 즉 사용자의 질의에 바로 그에 맞는 대답을 리턴한다. 좋은 대답은 일정 단어나 문맥 의미를 공유한다는 아이디어이다.

$$r =   \underset{t \in C}{\text{argmax}}\dfrac{q^Tt}{\| q \|  t \|}$$



각 케이스에서 단어나 임베딩에 대해 유사도 함수가 사용되는데 주로 코사인 유사도가 사용된다. 1)번이 좀 더 직관적인 알고리즘으로 보이지만 실제로는 2)번이 더 잘 동작한다. 대답을 위한 레이어가 추가적인 노이즈를 유발하기 때문이다.

IR 기반 챗봇은 사용자의 질의 $q​$ 외에도 추가적인 변수를 사용하고 full IR 랭킹 접근 방식을 사용해 확장할 수 있다. IR 기반 챗봇의 상업적 접근에는 Cleverbot과 Microsoft의 'XioaIce' (Little Bing, 小冰, 작은 얼음) 등이 있다.



### Sequence to sequence chatbots

다른 방법은 사용자의 이전 대화를 시스템의 턴으로 변환하는 것이다. Eliza의 머신러닝 버전이며, 질문을 대답으로 변환하는 것이다. 초기엔 구분 기반의 MT로 발전했으나 어려움을 겪는데, MT에서 입력 문장과 타겟 문장은 잘 매칭되지만 사용자 발화는 대답과 단어나 구문을 공유하지 않을 수도 있기 때문이다.

그 대신 seq2seq 모델을 이용한 변환 기반의 대답 생성 모델이 제안되었다. 기본 seq2seq는 실전을 위해 몇 가지 수정을 거쳐야 하는데 기본 seq2seq 모델은 예측 가능하지만 반복적인, 따라서 조금은 멍청한 "I'm OK"나 "I don't know" 등의 대답을 만들어내는 경향을 띠기 때문이다. 이는 목적 함수를 mutual information objective를 학습하도록 수정하거나 beam 디코더를 좀 더 다양한 대답을 유지하도록 수정하면 된다.

또 하나의 문제는 이전 대화의 긴 문맥의 모델링인데 이는 이전 몇 개의 정보를 요약하는 등의 계층적 모델을 사용해 모델이 이전의 대화를 보도록 하면 된다.

또한 SEQ2SEQresponse 모델은 단답만 생성해서 여러 턴에 걸쳐 응집된 대화를 생성하는 데는 좋지 않다. 이 점은 적대적 네트워크 같은 기술과 강화 학습을 통해 전체적으로 자연스러운 대화를 생성해내도록 함으로써 해결할 수 있다.



### Evaluating Chatbots

챗봇은 일반적으로 사람이 평가한다. BLEU는 챗봇의 대답과 인간의 대답을 비교하는 데 안 좋은 성능을 보인다. 이는 하나의 턴에 대응 가능한 대답이 매우 많기 때문이다. 대답의 경우의 수가 적고 어휘적으로 겹치는 부분이 있다면 word-overlap metrics가 가장 좋은 평가 성능을 보인다.

따라서 챗봇의 평가엔 인간의 개입이 필요하지만 자동 평가를 위한 모델이 제안되고 있다. ADEM은 인간이 그 적절성을 직접 평가한 대화 데이터를 이용했으며 대화 문맥과 시스템 대답의 단어로부터 태깅된 레이블을 예측하도록 학습했다. 또 하나의 패러다임은 adversarial evaluation이다. 이 패러다임의 아이디어는 기계가 만들어낸 대답과 인간이 만들어낸 대답 사이를 학습하는 것이다. 즉, 일반적인 GAN과 같이 시스템이 더 좋은 대답을 만들어낼 수록 그 성능은 개선되는 것이다.